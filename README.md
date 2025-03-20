# PentOps

Pentops is a highly opinionated application framework for event driven
microservices.

The two main concepts are 'O5' and 'J5'.

O5 defines the boundary between applications and infrastructure.

J5 is a schema and RPC definition language similar to google's ProtoBuf + gRPC.

The various repositories in the Pentops organization provide distinct components
and adapters of the overall framework.

The following is an introduction to the overall concepts of pentops, and is
deliberately vague to avoid bitrot. Individual components and schemas
should have more detailed documentation closer to their implementation.

## O5

A highly opinionated contract between infrastructure and applications.

An 'Application' is a docker image and o5 configuration file, usually 1:1 with a
git repository.

The o5 configuration file, e.g. ./ext/o5/app.yaml, defines the docker
containers, infrastructure resources, message subscriptions and targets, and
load balancer targets.

The details and documentation for configuration files is defined using a J5 /
proto schema in the o5-aws-deployer repo (TODO: This should be a standalone
repo).

- Applications log JSON to STDOUT or STDERR.
- Outside requests are routed as gRPC inbound calls.
- Communication between services is all via messaging, i.e. no direct
  inter-service communication.

Applications should be built against the o5 contract for infrastructure access.

O5 providers (e.g. o5-runtime-sidecar for AWS) handle the infrastructure
side of the contract. This makes applications infrastructure-independent, unless
their purpose is specifically tied to an infra-provided service which is not
exposed via O5.

### Runtime

A Runtime is a collection of docker container definitions, similar to a
docker-compose file.

Configuration of the runtime is passed in via environment variables.

Environment variables can be simple strings, environment dependent strings,
pre-defined secrets or access details for o5-provided infra components
(database, blobstore...)

When deployed, a runtime may have a 'sidecar' attached, or may have a central
adaptor, or a combination, to implement the infrastructure adaptors. The details
should be transparent to the application. The application need only know the
endpoints and credentials as provided in the environment variables. A measure
of success for the O5 project is the extent to which this remains true.

### J5 gRPC Interface

Application runtimes expose a gRPC endpoint with reflection enabled. 

Applications handle client requests and messages by implementing gRPC services
/ RPCs on this gRPC endpoint.

O5 providers interrogate the reflection endpoint at runtime to configure the
translation between the application's gRPC and the incoming HTTP request or
message.

The specifics of the gRPC definition and limitations are covered by J5.

### Routes (JSON/HTTP)

Runtimes configure 'Routes', which map a external HTTP prefixes to a runtime
containers, independently to the gRPC reflection provided by the application.

O5 config files must define the available endpoints for routing rules
independently of the reflection response. The existence of an endpoint in
reflection does not automatically register the service with the router or
message broker.

A default route configuration directs external client HTTP requests from the
router (e.g. ALB load-balancer) to the o5 provider (e.g. sidecar), where the J5
proxy handles authentication, some authorization, and translates the request to
gRPC before forwarding to the runtime.

Route definitions may bypass the J5 Proxy and directly expose HTTP endpoints.
This should only be used in cases where J5 is not a viable option, e.g., for
pre-existing or externally maintained contracts. An example of this in Pentops
is the 'registry' service which exposes a go module proxy. The interface for go
modules, unsurprisingly, does not conform to J5's opinionated standards.

### Message Handlers

Messages are defined in J5 schema files, producing gRPC services/methods with an empty
response body.

O5 config files must define the specific topics to subscribe to. The existence
of an endpoint in reflection does not automatically create a subscription in the
message broker.

Unlike Routes there is no 'bypass' option, however there are special message
definitions to take in events which do not confirm to the schema.

O5 providers manage subscriptions to the message broker, and handle errors as 'dead
letters' (e.g. the Dante handler). 

Messages are delivered to the application at least once in any order.

### PostgreSQL

O5 provides a number of tools for PostgreSQL.

O5 config files define postgres databases and environment variables to configura
the application. The environment variables are usually the `postgres://...`
style, but this is configurable in the file.

Applications do not need to implement any credential rotation code etc,
generally the environment variable points to a proxy which handles auth.

Applications may also provide migration container definitions, which causes the
o5 deployment process to include database migration.

### Publishing Messages

The primary method to publish messages uses the 'outbox' pattern in postgresql
tables.

Services without a database can publish messages using an 'adapter'
endpoint which exposes the same gRPC endpoints which the message handlers
implement.

O5 providers can be configured to monitor the outbox table of a postgres
database, and ensure messages are passed to the message broker before deleting
them from the table.

This allows the application to handle all business logic and publishing in
database transactions with rollback or commit support.

### Logging

Applications should log JSON formatted messages to STDOUT or STDERR. There are
currently no rules on the content of these messages, however a strong convention
is that messages should contain at a minimum a `"level"` (`DEBUG`, `INFO`,
`WARN`, `ERROR`) and `"timestamp"` in RFC3339 format.

### Blobstore (S3)

O5 config can define a "blobstore" resource which creates an S3 bucket. Yes,
specifically S3, this is a work in progress and is not yet abstracted. The
bucket's URL is passed in through environment variables as configured.


