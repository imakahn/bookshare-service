# book store service/app design exercise

[assignment](assignment.md)

## operations

- books can be added for rent on 'shelves'
- books can be rented
- books can be purchased - owner ID changes
- price can be set
- books can be removed
- full list of books on shelf can be queried
- records are kept of previous borrowers, owners - transaction log

## components

- [application architecture](architecture.md)

- [rationale behind design](DECISIONS.md)

- [database structure/queries](database.md)

- [openapi doc (w/schemas)](api.yml)

- [teaser p2p design](p2p-teaser.md)

## notes

- ISBN is in two formats, 10-digit and 13-digit. we can convert to 13-digit and store all in a single field in the DB, however the check (final) digit is calculated differently for both. validate check digit on input, before conversion
- rental price - percentage goes to owner, percentage captured by store
- backend can retrieve owner display details from /userinfo endpoint of OIDC (memoize!)
