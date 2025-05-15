Swimlane: External Network
  - User Email --> Microsoft Exchange Server (On-Premise)

Swimlane: Corporate Network
  - Firewall (between External and On-Premise AWS)
  - On-Premise AWS Environment:
    - Mulesoft Instance <--> Amazon Secret Service (Secure Credential Retrieval)
    - Mulesoft Instance --> GraphQL Service (HTTPS/TLS)
  - GraphQL Service --> Salesforce (HTTPS/TLS + OAuth 2.0)

Annotations (within Corporate Network):
  - Firewall: Inspects Traffic
  - Mulesoft: Processes Email, Security Checks
  - Amazon Secret Service: Secure Credentials
  - GraphQL Service: API Integration
