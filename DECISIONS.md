# Thoughts

A note on the decision-making process for the provided infrastructure design:

There were other designs that could have been produced for the purpose of an MVP that would have been simpler. Examples are:

## Alternatives

### Couchbase

pros:

- authentication/authorization, lending history, even views would largely be accounted for in one place
- _very_ easy to scale

cons:

- moving away from that architecture would essentially be a rewrite - however the data structures, if properly developed, would be transferrable
- limited provider selection -- self-host would require infrastructure more complex than current path

### Serverless

pros:

- simplest option in terms of deployability and the infrastructure we see
- much of infrastructure could remain the same - e.g. AWS services

cons:

- I have not yet found local (development) tools for this that I like
- platform lock-in (there are options on top of kubernetes - see below)

### Kubernetes

pros:

- tooling is very consistent, IDE support
- the most flexible option, best for platform agnosticism and future maneuverability
- easiest to maintain *once built* - infrastructure as config
- can integrate components like OIDC service (keyclock), database, etc
- can make services truly micro by removing responsibilities (mesh handles auth, policy, etc)

cons:

- if done at too early a stage, can end a project (and company)
- the line between what responsibilities should be done by the cloud service vs hosted within the kubernetes clusters is an art to define
- can host serverless applications (knative, etc)

## Conclusion

The infrastructure I have chosen for this project is perhaps not in vogue, but is very simple to define, explain, spin up quickly, and maintain. It can also handle a fair amount of traffic and change before it would be advantageous to look at another option - and provides a clear migration/extraction path in the case that occurs. My intention here is to express a measured, incremental approach to building an architecture that accounts for a given stage of company/project.
