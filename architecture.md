# Architecture

## data flow for borrowing book

1. request to api
2. payment authorization takes place
3. app makes sql queries to update status of book to new borrower
4. response returned

## Infrastructure

```text
  ┌───────────────┐       ┌──────────────────────┐
  │               │       │                      │
  │   Payment    │       │   User Management    │
  │   Processor   │   ┌───►   OIDC Provider      │
  │               │   │   │                      │
  └▲──────────────┘   │   └──────────────────────┘
   │                  │
   │                  │
   │  ┌───────────────┘
   │  │
┌──┼──┼───────────────────────────────────────────────┐
│  │  │                 VPC                           │
│  │  │                                               │
│  │  │                                               │
│  │  │    ┌───────────────────────────┐              │
│  │  │    │            VM             │              │
│  │  │    │                           │              │
│  │  │    │   ┌───────────┐           │              │
│  │  │    │   │ container │           │              │
│  │  │    │   ├─┬────┬────┤ ┌─────┐   │              │
│  │  │ ┌──┼───┼┼│    │┼┼┼┼┼─►nginx◄───┼──┐           │
│  │  │ │  │   ├─┴────┴────┤ └─────┘   │  │   ┌────┐  │
│  │  │ │  │   │ container │           │  │   │    │  │
│  │  └─┤  │   └───────────┘           │  │   │    │  │
│  │    │  │                           │  │   │    │  │
│  │    │  └───────────────────────────┘  │   │    │  │    ┌─────────────┐
│  │    │                                 │   │ L  │  │    │             │
│  │    │                                 │   │ B  │  │    │             │
│  └────┤  ┌───────────────────────────┐  ├───┼────┼──┼────┤     UI      │
│       │  │            VM             │  │   │    │  │    │             │
│       │  │                           │  │   │    │  │    │             │
│    ┌──┤  │   ┌───────────┐           │  │   │    │  │    └─────────────┘
│    │  │  │   │ container │           │  │   │    │  │
│    │  │  │   ├─┬────┬────┤ ┌─────┐   │  │   └────┘  │
│    │  └──┼───┼┼│    │┼┼┼┼┼─►nginx◄───┼──┘           │
│    │     │   ├─┴────┴────┤ └─────┘   │              │
│    │     │   │ container │           │              │
│    │     │   └───────────┘           │              │
│    │     │                           │              │
│    │     └───────────────────────────┘              │
│    │                                                │
│    │                                                │
│    │        ┌──────────────────┐                    │
│    │        │      RDS         │                    │
│    │        │                  │                    │
│    │        │  ┌────────────┐  │                    │
│    │        │  │   Master   │  │                    │
│    │        │  └────────────┘  │                    │
│    └────────┼──►               │                    │
│             │  ┌────────────┐  │                    │
│             │  │   Minion   │  │                    │
│             │  └────────────┘  │                    │
│             │                  │                    │
│             └──────────────────┘                    │
│                                                     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

The infrastructure is composed of two VMs, an RDS cluster, and a LB within a VPC. External resources are the user management/authentication system and the payment processor.

The two VMs are set up in a blue/green arrangement with one serving at a given time. The second VM serves both as failover and as the target instance for the next deploy (these swap each deploy). Within each VM are two dockerized instances of the application managed by systemd and balanced by an nginx instance.

## Deployment

```text
cicd ----->
            trigger on commit to master
                                        ----> image build
                                                           ----> VM2 deploy
                                        ----> test  
                                                           ----> LB switch
                                        ----> VM2 <--> VM1 swap
```

Deployment is managed by a script set on a trigger. An image is built and deployed to the VM instance that is currently out of the LB. Once the service is confirmed to be live, the LB switches over, and VM2 becomes VM1.

## Application Design

The application design is a simple backend-for-frontend arrangement, ready to be extracted into dedicated backend services separate from external API servicing in the future. Dependencies are isolated into modules which provide internal APIs to the rest of the service, enabling our database, payment processor, etc. to be changed in the future with only one area requiring change.

The Payments module, for instance, could expose purchase and borrow functions, with all the logic to execute on that done in isolation.

The management of users will vary - in this design that responsibility lies outside of the application (such as by using Google's system as opposed to an Okta instance with internal provisioning).

## Error handling

- book reservation race condition
  - we examine the latest status entry on insert (stored procedure OR queue-driven service)
  - IF status is marked as unavailable, someone else got there first
    - reject
  - succeed

## Authn/Authz

- oidc tokens - ID token for UI (name, handle, etc)
- authz based on ID for operation - if they are the owner, etc
- future -- could be done in mesh rather than in service
