
A pod running in AWS (EKS or Kubernetes-on-EC2) uses its Kubernetes ServiceAccount to authenticate with Microsoft Entra ID using OIDC federation, without any user interaction.
c
🔧 High-Level Setup Steps
1. Ensure Kubernetes is set up with OIDC Issuer
For EKS, OIDC is already configured, and the OIDC issuer URL is available from:

bash
Copy
Edit
aws eks describe-cluster --name your-cluster-name --query "cluster.identity.oidc.issuer" --output text
For self-managed Kubernetes, you must configure the API server with the --service-account-issuer and --service-account-signing-key-file flags.

2. Register App in Microsoft Entra ID (Azure AD)
Go to Azure portal → App registrations → New registration.

Under Redirect URI, choose “Web” and leave it blank or enter a dummy if required.

After creation, note the Application (client) ID and Directory (tenant) ID.

3. Expose the AWS OIDC issuer to Entra ID
Go to App registration → [Your App] → Certificates & Secrets → Federated credentials.

Click + Add credential, and choose "Workload identity federation".

Provide the following:

Issuer: The AWS OIDC Issuer URL (https://oidc.eks.region.amazonaws.com/id/EXAMPLED539D4633E53DE1B716D3041E)

Subject: Should match the Kubernetes service account format:

ruby
Copy
Edit
system:serviceaccount:<namespace>:<serviceaccount-name>
Audience: Typically sts.amazonaws.com (or whatever audience is expected in the token by Entra).

4. Configure Pod to Use the ServiceAccount
Define a Kubernetes ServiceAccount with an annotation linking to the OIDC provider:

yaml
Copy
Edit
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/my-app-role
5. Inside the Pod: Use Microsoft Identity Platform Token Endpoint
You can now fetch a token from Azure AD using the federated token from the projected service account token file:

Inside the pod, the token is available at:

swift
Copy
Edit
/var/run/secrets/kubernetes.io/serviceaccount/token
Use this JWT to get a token from Azure:

http
Copy
Edit
POST https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id=<azure-client-id>&
scope=<your-api-scope or default like https://graph.microsoft.com/.default>&
grant_type=urn:ietf:params:oauth:grant-type:token-exchange&
requested_token_type=urn:ietf:params:oauth:token-type:access_token&
subject_token_type=urn:ietf:params:oauth:token-type:jwt&
subject_token=<JWT-from-SA-token-file>
✅ Summary
Step	Description
1	Get OIDC issuer from EKS or configure your own in k8s
2	Register an app in Entra ID
3	Add federated credential in the app using OIDC issuer and serviceaccount subject
4	Assign a k8s serviceaccount to your pod
5	Use the projected token inside the pod to exchange for Entra access token (via Azure AD token endpoint)
