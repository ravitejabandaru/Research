@startuml
actor User
participant "Microsoft Exchange\nServer (On-Premise)" as Exchange
participant "Mulesoft\n(On-Premise AWS)" as Mulesoft
participant "Amazon\nSecret Service" as SecretsManager
participant "GraphQL\nService" as GraphQL
participant Salesforce

User -> Exchange: Sends Email
Exchange -> Mulesoft: Retrieves Email (IMAPS/POP3S)
activate Mulesoft
Mulesoft -> SecretsManager: Requests Exchange Credentials
activate SecretsManager
SecretsManager --> Mulesoft: Returns Exchange Credentials (Securely)
deactivate SecretsManager
Mulesoft -> Mulesoft: Processes Email & Attachments\n(Malware Scan, DLP, Transformation)
Mulesoft -> GraphQL: Sends Processed Data (HTTPS/TLS)
activate GraphQL
GraphQL -> Salesforce: Creates Case (HTTPS/TLS + OAuth 2.0)
activate Salesforce
Salesforce --> GraphQL: Acknowledges Case Creation
deactivate Salesforce
GraphQL --> Mulesoft: Acknowledges Data Received
deactivate GraphQL
@enduml
