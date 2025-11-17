graph TD
    A[Client App] -->|GraphQL Query| B(GraphQL API Gateway / Server);
    B -->|IPC: Internal HTTP/gRPC/RPC| C[Microservice A (Users)];
    B -->|IPC: Internal HTTP/gRPC/RPC| D[Microservice B (Orders)];
    B -->|IPC: Internal HTTP/gRPC/RPC| E[Microservice C (Inventory)];
    C --> F((DB A));
    D --> G((DB B));
    E --> H((DB C));
    B -->|GraphQL Result| A;

    subgraph Distributed System (Microservices)
        C
        D
        E
        F
        G
        H
    end

    style B fill:#f9f,stroke:#333,stroke-width:2px;
    style A fill:#ccf,stroke:#333,stroke-width:2px;
