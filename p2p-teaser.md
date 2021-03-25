# Notes

## Components

### Link - Peer Discovery

* provides the initial list of peers to client
* can be in link form for easy distribution - such as a magnet link (tracker url property)
* structure:

### Peer List

* json document - tuples with peer locations and weights
* structure:

### Hash - Chain Validation

* stored and retrieved separately from chain
* structure:

### Chain

* json document
* validation/consistency operation via hash, json schema
* structure:

### Events

* json document
* structure:
* validation functions by peers - json schema and against rules - price as set, agreements, etc

### Consensus algo for conflicts

* first - append all versions
* then decide which version to use and store that property as reference
* trusted peer wins
* time received timestamp from trusted peer wins
* decision may be made within small group but needs to be re-evaluated as spread throughout ecosystem of peers
* subnets?
* need at least three peers for writes to be accepted

### Trust algo

* weighted peers
* age of peers

## Caveats

## References

* https://www.oreilly.com/library/view/couchdb-the-definitive/9780596158156/ch02.html
* https://en.wikipedia.org/wiki/Leader_election
* https://en.wikipedia.org/wiki/Atomic_broadcast
* https://en.wikipedia.org/wiki/State_machine_replication
* https://en.wikipedia.org/wiki/Magnet_URI_scheme
* http://web.mit.edu/6.033/2005/wwwdocs/quorum_note.html