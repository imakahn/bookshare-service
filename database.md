# Data store design

This design aims to be as simple as possible while meeting all requirements and remaining flexible to future changes.

There are three tables--

- book
- book_state
- user

The *book* table stores only the properties of a given copy of a book that will not change, with the exception of a state property, that points to the current version of its state.

The *book_state* table serves both as transactional log and source of all mutable data for a given copy of a book. When a book is borrowed, a new row is inserted that has all the same properties as before, but with an updated borrower_id. Same goes for ownership transfer (purchase). Rate for borrowing and price for purchase are also set here, etc.

To find all previous borrowers, owners, rates etc. for a book, queries can be made against this table. For optimization purposes, such as listing all books under a given owner and listing all books in the database, the current state is referenced via the *book* table.

Lastly we have the *user* table. This serves as a bridge between the database and external user management. By normalizing these properties, rather than storing them directly in the *book_state* table, we ease management. Triggers can do associated cleanup when changes are made to the *user* table. Provisioning would need to be configured between this and the user management service. It is the responsibility of the backend service to integrate data from the user management service when generating responses.

## table structure

```sql
-- TODO mysql->postgres to avoid mysql UUID insanity
-- TODO indexing

CREATE TABLE book (
  id BINARY(16) DEFAULT (UUID_TO_BIN(UUID())) PRIMARY KEY,
  isbn CHAR(13) NOT NULL,
  title VARCHAR(64) NOT NULL,
  cover MEDIUMBLOB,
  state INT,

  FOREIGN KEY (state) REFERENCES book_state(id)
);

CREATE TABLE book_state (
  id INT PRIMARY KEY,
  book_id BINARY(16),
  owner_id BINARY(16),
  borrower_id BINARY(16),
  price DECIMAL(5, 2),
  rate DECIMAL(4, 2),
  archived BOOLEAN NOT NULL DEFAULT 0,
  
  FOREIGN KEY (book_id) REFERENCES book(id)
  FOREIGN KEY (owner_id) REFERENCES user(id)
  FOREIGN KEY (borrower_id) REFERENCES user(id)
);

CREATE TABLE user (
  id BINARY(16) DEFAULT (UUID_TO_BIN(UUID())) PRIMARY KEY,
  openid_id CHAR(255) -- 'sub' property in ID token

  -- TODO on delete trigger - clean book_state w/stored procedure (will either remove (archive) book entirely or set to returned/missing)
);
```

## example queries

- TODO redo to find all books under owner, search states by owner_id to get list of book ids. search states by book ids and filter out all but newest per book, then filter by owner

```sql
-- set new owner

START TRANSACTION;

SELECT `state` FROM book INTO @current WHERE id = 'c4b1ec40-8d28-11eb-8dcd-0242ac130003';

INSERT INTO book_status (book_id, owner_id, borrower_id, price, rate)
SELECT book_id,
  UUID_TO_BIN('adcd0f36-8d06-11eb-8dcd-0242ac130003'), -- provided by API query
  borrower_id,
  price,
  rate
FROM book_id
WHERE id = @current;

SELECT MAX(id) + 1 FROM book_state INTO @current;

UPDATE book SET status = @current

COMMIT;

-- find all books under owner

SELECT book.id,
  book.isbn,
  book.title,
  book.cover,
  book_state.id,
  book_state.owner_id,
  book_state.borrower_id,
  book_state.price,
  book_state.rate
FROM book
  JOIN book_state ON book.id = book_state.book_id
WHERE book_state.archived <> 0
  AND book_state.owner_id = 'adcd0f36-8d06-11eb-8dcd-0242ac130003';

-- find all previous borrowers of book

SELECT book_state.borrower_id,
FROM book_state
WHERE book_state.book_id = 'c4b1ec40-8d28-11eb-8dcd-0242ac130003';
```

## change strategy

### non-breaking change

1. run migration

### breaking change

1. run migration to ADD new structures
2. deploy application to READ from new/old structures and WRITE to new structures
3. run migration to MOVE remaining data from old to new structures
4. run migration to REMOVE old structures
