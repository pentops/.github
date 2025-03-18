# PentOps

A clearly defined, highly opinionated contract between platform and applications.

- Applications are docker images
- Applications communicate ONLY through gRPC
- Outside requests are routed as JSON/HTTP or gRPC calls
- Communication between services is messaging, over gRPC
- Sidecars and other adapters adapt the application's gRPC endpoints to the infra (messaging etc)


![Pentops - Page 2](https://user-images.githubusercontent.com/1665328/236635883-bd166b1b-6332-4e57-8a2e-472096bacd2c.png)
