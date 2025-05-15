graph LR
    subgraph Corporate Network
        direction LR
        A[User Email] --> B(Microsoft Exchange\nServer (On-Premise));
        B --> C{{Mulesoft\n(On-Premise AWS)}};
        C --> D(Amazon\nSecret Service);
        D -- Provides Credentials --> C;
        C -- Processes Email --> E{{GraphQL\nService}};
        E -- Creates Case --> F(Salesforce);
        style A fill:#f9f,stroke:#333,stroke-width:2px
        style B fill:#ccf,stroke:#333,stroke-width:2px
        style C fill:#f9f,stroke:#333,stroke-width:2px
        style D fill:#ccf,stroke:#333,stroke-width:2px
        style E fill:#f9f,stroke:#333,stroke-width:2px
        style F fill:#ccf,stroke:#333,stroke-width:2px
    end
    G[Corporate Firewall]
    G -- Allows Internal Traffic --> B
    G -- Allows Internal Traffic --> C
    G -- Allows Internal Traffic --> E
