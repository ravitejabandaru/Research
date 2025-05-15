  1. User -> Microsoft Exchange Server (On-Premise): Sends Email
  2. Microsoft Exchange Server (On-Premise) -> Mulesoft (On-Premise AWS): Retrieves Email (IMAPS/POP3S)
     - Activation of Mulesoft
  3. Mulesoft (On-Premise AWS) -> Amazon Secret Service: Requests Exchange Credentials
     - Activation of Amazon Secret Service
  4. Amazon Secret Service --> Mulesoft (On-Premise AWS): Returns Exchange Credentials (Securely)
     - Deactivation of Amazon Secret Service
  5. Mulesoft (On-Premise AWS) -> Mulesoft (On-Premise AWS): Processes Email & Attachments (Malware Scan, DLP, Transformation)
  6. Mulesoft (On-Premise AWS) -> GraphQL Service: Sends Processed Data (HTTPS/TLS)
     - Activation of GraphQL Service
  7. GraphQL Service -> Salesforce: Creates Case (HTTPS/TLS + OAuth 2.0)
     - Activation of Salesforce
  8. Salesforce --> GraphQL Service: Acknowledges Case Creation
     - Deactivation of Salesforce
  9. GraphQL Service --> Mulesoft (On-Premise AWS): Acknowledges Data Received
     - Deactivation of GraphQL Service
     - Deactivation of Mulesoft
