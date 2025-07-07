We are considering the following resilience strategies for the Email-to-Case integration using Microsoft Graph API and MuleSoft Runtime on AWS EKS. Based on POC results and compliance decisions (e.g., whether we can move emails), we will finalize the best-fit option.

⸻

Option 1: Message-ID Tracking via Redis
	•	What it is: Store processed InternetMessageId in Redis with TTL to prevent duplicate processing.
	•	Steps:
	1.	After successful processing, store the Message-ID as a key in Redis (processed:<messageId>).
	2.	Set TTL (e.g., 7 days) for automatic cleanup.
	3.	Before processing each email, check Redis to skip duplicates.
	•	Resilient Against: Pod restarts, retries, and out-of-order delivery.
	•	Infra Required: Redis (managed like ElastiCache or containerized).

⸻

Option 2: Log Processed Emails to S3
	•	What it is: Append processed email metadata (ID + timestamp) to S3 (as JSON log).
	•	Steps:
	1.	After processing, write the Message-ID + timestamp to a log file or object in S3.
	2.	Optionally, check this log before processing (or use Athena/Glue for queries).
	3.	S3 acts as a long-term audit trail.
	•	Resilient Against: Data loss, long-term duplication, audit requirements.
	•	Infra Required: S3 bucket with lifecycle policies.

⸻

Option 3: Use Salesforce External ID
	•	What it is: Store InternetMessageId in a custom External ID field on the Case object.
	•	Steps:
	1.	Set the External ID on Case insert.
	2.	If the same message is received again, insertion fails or is skipped.
	•	Resilient Against: Inserting duplicates into Salesforce.
	•	Limitation: Does not prevent duplicate processing in MuleSoft — only blocks insert into Salesforce.

⸻

Option 4: Database (RDS or DocumentDB)
	•	What it is: Track email processing status in a relational or NoSQL DB.
	•	Steps:
	1.	Insert record per email with ID, timestamp, status.
	2.	Check this DB before processing.
	3.	Can extend schema to store retry status, errors, etc.
	•	Resilient Against: Complex processing logic, retry flows, partial failures.
	•	Infra Required: RDS or DocumentDB, schema management.

⸻

Option 5: Local MuleSoft Object Store (for POC only)
	•	What it is: Use built-in MuleSoft object store to store processed Message-IDs temporarily.
	•	Steps:
	1.	Write processed IDs with TTL in MuleSoft’s object store.
	2.	Lookup before processing.
	•	Note: Not shared across pods; not recommended for production. Just for basic testing.

⸻

Option 6: Move Email to ‘Processed’ Folder (Preferred if Allowed)
	•	What it is: After successful processing, move email to another folder using Microsoft Graph.
	•	Steps:
	1.	Poll only the Inbox folder.
	2.	On success, move the email to /Processed folder via Graph API.
	3.	Next cycle only fetches unprocessed emails still in Inbox.
	•	Resilient Against: All failure modes; no need for Redis/S3/DB.


Option 7: Delta Query Support (If Mailbox Supports It)
	•	What it is: Use Graph API Delta Queries to get only changed messages.
	•	Steps:
	1.	Use /delta endpoint on first call to get state token.
	2.	On next poll, pass that token to receive only new/delta messages.
	•	Resilient Against: Over-fetching, reduces data volume.
	•	Note: Only supported for some mailbox types (needs confirmation).

⸻

Option 8: In-Memory LRU Cache (within MuleSoft Pods)
	•	What it is: Store last N processed IDs in memory inside each pod.
	•	Steps:
	1.	Keep a small (~1,000 items) LRU map per pod.
	2.	Skip emails already seen in that cache.
	•	Resilient Against: Short-term duplicates, retries.
	•	Limitation: Not resilient to pod crashes; use only with another durable option.

⸻

Option 9: Dead Letter Queue (DLQ) or Retry Store
	•	What it is: Store failed email message IDs for later triage.
	•	Steps:
	1.	On permanent failure, log message ID and reason to S3 or DLQ.
	2.	Provide a reprocessing mechanism for DLQ entries.
	•	Resilient Against: Message loss during unexpected failures.
	•	Infra Required: S3 or database-based DLQ storage.

⸻

Option 10: No State Tracking (POC-Only Quick Test)
	•	What it is: Just poll based on timestamp and process all matching emails.
	•	Steps:
	1.	Use Graph $filter=receivedDateTime queries.
	2.	Process all returned emails.
	•	Limitation: High chance of duplicates unless mailbox is small and testing is isolated.

⸻

✅ Next Steps

During the POC, we will implement and evaluate:
	•	Option 1 (Redis) for fast duplicate detection
	•	Option 2 (S3) for audit durability
	•	Option 6 (Email Move) if allowed by InfoSec
	•	Others depending on mailbox type and performance

Final selection will be made based on:
	•	Message volume
	•	Graph API limits
	•	Compliance rules (e.g., move/delete email restrictions)
	•	Recovery needs

=================









=

⸻

Description:

This ticket seeks ARGW approval for integrating MuleSoft Runtime deployed on AWS xCORE EKS (JPMC internal environment) with cloud-hosted Microsoft 365 shared mailboxes using Microsoft Graph API. Authentication is secured using Azure Entra ID (Azure AD) via Credential-to-Credential (C2C), adhering to JPMC’s established identity federation standards.

⸻

Context:

The integration is required to automate the ingestion of emails and attachments from Microsoft 365 shared mailboxes. The data extracted will feed internal systems such as Case Manager and Document Manager via MuleSoft and the Florence platform. This approach aligns with JPMC’s cloud migration strategy and security compliance framework.

⸻

Proposed Architecture:
	•	Microsoft 365 Exchange Online: Source system hosting shared mailboxes.
	•	Microsoft Graph API: Standardized REST API to access mailbox content.
	•	Azure Entra ID (Azure AD): OAuth 2.0 service for authentication, using JWT tokens validated via Internet-Facing OIDC Metadata.
	•	JPMC Credential-to-Credential (C2C) Service: Provides secure JWT token exchange leveraging AWS IAM roles.
	•	AWS Security Token Service (STS) and IAM Role: Issue short-lived tokens to ensure secure, temporary credentials.
	•	AWS Secrets Manager: Secure repository for configuration details and initial authentication setup.
	•	Network Edge Proxy and ExpressRoute: Ensures secure, compliant communication paths between AWS xCORE and Azure Cloud.
	•	MuleSoft Runtime (on AWS xCORE EKS): Processes email data retrieved via Graph API and integrates downstream services.
	•	Florence Egress Gateway: Routes securely to internal services like Case Manager and Document Manager (previously approved integration).

⸻

Security:
	•	Utilizes JPMC’s approved C2C identity federation framework for secure authentication without static secrets.
	•	Azure Entra ID verifies tokens through trusted OIDC metadata endpoints.
	•	All external communication is encrypted via TLS over HTTPS and controlled through JPMC Network Edge Proxy and private ExpressRoute.

⸻

Sequence Diagram:
	1.	MuleSoft Runtime assumes AWS IAM role.
	2.	MuleSoft requests JWT token via JPMC C2C.
	3.	C2C Service issues JWT token validated by Azure Entra ID using Internet-Facing OIDC metadata.
	4.	MuleSoft obtains OAuth token from Azure Entra ID.
	5.	MuleSoft retrieves email data from Microsoft Graph API.
	6.	MuleSoft processes emails, extracting attachments and metadata.
	7.	MuleSoft sends processed data to internal systems (Case Manager & Document Manager) through Florence Egress Gateway.

⸻

Decision Required:

ARGW approval of the proposed integration architecture for MuleSoft Runtime to securely integrate with Microsoft 365 via Graph API and Azure Entra ID C2C authentication, adhering fully to JPMC security and compliance guidelines.

participant MuleSoft Runtime
participant AWS IAM Role
participant AWS STS
participant JPMC C2C Service
participant Azure Entra ID
participant Microsoft Graph API
participant Florence Egress Gateway
participant Case & Document Manager

MuleSoft Runtime -> AWS IAM Role: Assume IAM Role
AWS IAM Role -> AWS STS: Request temporary token
AWS STS --> AWS IAM Role: Return STS Token
AWS IAM Role --> MuleSoft Runtime: Provide STS Token
MuleSoft Runtime -> JPMC C2C Service: Request JWT Token using STS Token
JPMC C2C Service --> MuleSoft Runtime: Provide JWT Token
MuleSoft Runtime -> Azure Entra ID: Request OAuth Token using JWT Token
Azure Entra ID --> MuleSoft Runtime: Provide OAuth Token
MuleSoft Runtime -> Microsoft Graph API: Call Graph API (Fetch Emails)
Microsoft Graph API --> MuleSoft Runtime: Return Emails & Attachments
MuleSoft Runtime -> Florence Egress Gateway: Route extracted email data
Florence Egress Gateway --> Case & Document Manager: Forward processed data


participant MuleSoft as MS
participant AWS_IAM_Role as IAM
participant AWS_STS as STS
participant JPMC_C2C_Service as C2C
participant Azure_Entra_ID as AzureAD
participant Microsoft_Graph_API as GraphAPI
participant Florence_Egress_Gateway as FlorenceGW
participant Case_and_Document_Manager as CaseDocMgr

MS -> IAM: Assume IAM Role
activate IAM

IAM -> STS: Request Temporary Token
activate STS
STS --> IAM: Return STS Token
deactivate STS

IAM --> MS: Provide STS Token
deactivate IAM

MS -> C2C: Request JWT Token (STS Token)
activate C2C
C2C --> MS: Provide JWT Token
deactivate C2C

MS -> AzureAD: Request OAuth Token (JWT)
activate AzureAD
AzureAD --> MS: Return OAuth Token
deactivate AzureAD

MS -> GraphAPI: Fetch Emails & Attachments
activate GraphAPI
GraphAPI --> MS: Return Emails & Attachments
deactivate GraphAPI

MS -> FlorenceGW: Send Processed Data
activate FlorenceGW

FlorenceGW -> CaseDocMgr: Forward Data
activate CaseDocMgr
CaseDocMgr --> FlorenceGW: Acknowledge Receipt
deactivate CaseDocMgr

FlorenceGW --> MS: Confirm Forwarding
deactivate FlorenceGW

============================

The solution leverages Microsoft Entra ID and the JPMC Credential-to-Credential (C2C) identity federation model to securely connect the MuleSoft Runtime (hosted in AWS xCore EKS) to Microsoft Graph API for accessing shared mailboxes in Exchange Online.

The authentication and authorization flow is handled entirely through short-lived, non-human-accessible credentials with automatic key rotation, ensuring high compliance with enterprise and regulatory standards.

Credential-to-Credential (C2C) Flow:
	1.	An application running on AWS (MuleSoft Runtime) calls the C2C Service to exchange its workload’s IAM role identity token for a security token trusted by Entra ID.
	2.	The C2C service authenticates the AWS IAM role using AWS Security Token Service (STS).
	3.	MuleSoft passes the C2C token to Microsoft Entra ID STS to receive an access token used for calling the protected Microsoft Graph API.
	4.	Entra ID validates the C2C token using cryptographic signing keys that are rotated automatically and never exposed to humans, then issues a short-lived access token.
	5.	MuleSoft uses this Entra access token to authenticate to the Microsoft Graph API.
	6.	Microsoft Graph API (an Entra-protected application) validates the access token using Entra’s signing keys, granting access securely.

All credentials are handled securely using:
	•	AWS Secrets Manager (to store the C2C client ID and secret).
	•	xCore EKS AuthorizationPolicy to control outbound traffic and integrate with Istio gateways.
	•	ExpressRoute and Network Edge Proxy to route traffic securely to Azure, bypassing public internet exposure.

This approach ensures that no long-lived secrets are used, credentials are rotated automatically, and access tokens are strictly scoped and short-lived.
