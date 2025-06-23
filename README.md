

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