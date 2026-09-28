## With OIDC (Federated Credentials), 
GitHub and Azure trust each other directly through digital certificates.
#### Part 1: On the Azure (Entra ID) Side
1.` Register an App in Entra ID`: You create an App Registration (which gives you an App ID / Client ID).  
2. Create a Federated Credential (OIDC Trust): Instead of creating a password/client secret, you tell Azure:  

> `"Hey Azure, if GitHub comes to you with a token saying it is running code from my specific repository (raghvendrashrinet/Azure-Key-Vault) on the main branch, trust it."`

3. Assign Permissions: You give that App Registration access (e.g., Key Vault Secrets User) to your Key Vault so it can read secrets.

#### Part 2: On the GitHub Side
1. Store Azure Identifiers in GitHub Secrets / Variables
    GitHub repository Settings $\rightarrow$ Secrets and variables $\rightarrow$ Actions and add these 3 variables
     - `AZURE_CLIENT_ID`: The App (client) ID of your Azure App Registration.
     - `AZURE_TENANT_ID`: Your Azure Active Directory Tenant ID.
     - `AZURE_SUBSCRIPTION_ID`: Your Azure Subscription ID.
  
2. Configure Your GitHub Actions Workflow File (`.github/workflows/deploy.yml`)
   1. `Grant OIDC Permission`: Tell GitHub to issue an OIDC token when the workflow runs:
      ```yaml
      permissions:
        id-token: write  # Allows GitHub to request an OIDC token from Azure
        contents: read
      ```
   2.` Authenticate with azure/login@v2`: Use the official Azure action to exchange that short-lived token:
      ```yaml
      - name: Authenticate to Azure via OIDC
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    ```
      ***GitHub Generates a Short-Lived OIDC Token(JWT)***
