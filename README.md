

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
