# PentOps

A clearly defined, highly opinionated contract between platform and applications.

- Applications are docker images
- Applications communicate ONLY through gRPC
- Outside requests are routed as JSON/HTTP or gRPC calls
- Communication between services is messaging, over gRPC
- Sidecars and other adapters adapt the application's gRPC endpoints to the infra (messaging etc)


