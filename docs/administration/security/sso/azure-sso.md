# Single SignOn (SSO) - Azure Active Directory

PagerDuty Process Automation can be configured to use Azure Active Directory for authentication by registering a new application in Azure Active Directory and configuring PagerDuty Process Automation to use it.

## Configuring Azure Active Directory

### Create a new app registration

The first thing to do is create a new application registration in Azure.

![app reg link1](~@assets/img/sso-azure-01-appreg1.jpg)

1. Begin by opening to Azure Active Directory
2. Select **"App registrations"** on the left
3. Select **"+ New registration"** near the top

![app reg link2](~@assets/img/sso-azure-02-appreg2.jpg)

       (Redo: wrong URL name)

1. Enter **"PagerDuty Process Automation On-Prem"** for the Name (or any name you like)
2. Leave the default selection for the Support account types
3. Select **"Web"** for the Redirect URI type
4. Enter **"https://paop.company.com/login/oauth2/code/azure"** for the Redirect URI<br>
    Note: This URL should the PagerDuty Process Automation URL
5. Select **"Register"** at the bottom 

### Add the required application permissions

Next, add the required permissions in Azure.

![app perm link1](~@assets/img/sso-azure-03-apiperm1.jpg)

1. Select **"API permissions"** on the left
2. Select **"+ Add a permission"**
3. Select **"Microsoft Graph"** as the permission type
4. Select **"Delegated permissions"**
5. Select **"email"** to enable the permission
6. Select **"openid"** to enable the permission
7. Select **"profile"** to enable the permission
8. Select **"Add permission"** at the bottom

![app perm link2](~@assets/img/sso-azure-04-apiperm2.jpg)

1. Select **"API permissions"** on the left
2. Select **"+ Add a permission"**
3. Select **"Microsoft Graph"** as the permission type
4. Select **"Application permissions"**
5. Enter **"Directory.Read.AlL"** in the search box
6. Select **"Directory.Read.All"** under Directory to enable the permission
7. Select **"Add permission"** at the bottom

### Create the Application Secret

Next, create an application secret (ID & passsword) that will be used in the Process Automation configuration.  Note, if you lose the secret value/password, you can delete the existing secret and create a new one.

![app secret link1](~@assets/img/sso-azure-05-secret1.jpg)

1. Select **"Certificates & secrets"** on the left
2. Select **"+ New client secret"**
3. Enter **"PagerDuty Process Automation On-Prem"** for the Description (or any name you choice)
4. Select **"Add"** at the bottom

![app secret link2](~@assets/img/sso-azure-06-secret2.jpg)

1. Copy the **Value** and store it someplace. You will use it as the clientSecret (password) when configuring Process Automation On-Prem. (Hint: use the Copy to clipboard button)
2. Copy the **Secret ID** and store it someplace. You will use it as tge clientID when configuring Process Automation On-Prem. (Hint: use the Copy to clipboard button)

### Get the **"Directory (tenant) ID"**

Last, capture the Directory (tenant) ID to use in configuring Process Automation.

![app dirid link](~@assets/img/sso-azure-07-dirid.jpg)
    
1. Click **"Overview"** on the left
2. Copy the **"Directory (tenant) ID"** and store it someplace.  You will use it in the URL when configuring Process Automation On-Prem. (Hint: use the Copy to clipboard button)


## Configure PagerDuty Process Automation On-Prem to use Azure Active Directory for Authentication

Azure Active Directory integration is configured within the `rundeck-config.properties` file.  Below are the required and optional settings to be added.  Be sure to fill in your Directory (tenant) ID, Secret ID and Value that you previously saved. After making the changes to the config file, a server restart is required.

```
  
rundeck.security.oauth.enabled=true  
rundeck.sso.loginButton.enabled=true  
rundeck.sso.loginButton.title=Login with Azure  
rundeck.sso.loginButton.url=oauth/azure  
rundeck.security.oauth.azure.autoConfigUrl=https://login.microsoftonline.com/<DIRECTORY_TENANT_ID>/v2.0  
rundeck.security.oauth.azure.clientId=<SECRET_ID>  
rundeck.security.oauth.azure.clientSecret=<SECRET_VALUE>  
rundeck.security.syncOauthUser=true  
  
# Map Azure groups by default (can be commented out if not mapping group permissions)  
framework.plugin.UserGroupSource.AzureGroupSource.enabled=true  
rundeck.security.oauth.azure.scope=openid email profile https://graph.microsoft.com/Directory.Read.All
  
# Map Azure user detail attributes  
rundeck.ssoSyncAttribNames.firstname=given_name  
rundeck.ssoSyncAttribNames.lastname=family_name  
rundeck.ssoSyncAttribNames.email=preferred_username  
  
```

### Important: First Login Approval

Upon first login to Process Automation On-Prem using Azure SSO an Azure Admin level user will need to consent to the `Directory.Read.All` permission. Make sure to click the checkbox that asks to consent for the _whole organization_.

## Note: Azure Groups

By default, Microsoft does not send usable group information in the SSO token. To get a user’s groups the plugin uses the MS Graph API to get user/group information. Using this requires additional API permissions that were setup in the App Registration.  The default config file enables group mapping.

## Note: firstname, lastname and email attribute mapping

If your Azure Active Directory attributes are non-standard, you can specify the correct attribute values to use.  You can verify the values by selecting the person icon in the upper right corner after logging in, and selecting **Profile**.  If you see **NOT SET** for any fields, you will need to specify the correct attribute mappings.

## Note: Debugging tips

If you are having trouble with the Azure integration, these additional config file entries will generate helpful debugging information.  Adding the following lines to the **log4j.properties** file will produce additional debugging output in the services.log file.

```
#SSO ADDTL LOGGING
logger.oauth2.name=org.springframework.security.oauth2  
logger.oauth2.level=debug  
logger.oauth2.additivity=false  
logger.oauth2.appenderRef.stdout.ref=STDOUT  
```

### Note: Azure Government

The Azure Groups plugin uses the MS Graph API endpoint to gather the groups.  By default it will use the endpoint `https://graph.microsoft.com/v1.0/users/`.  For some Azure environments (Government, etc.) a different endpoint may be needed. ([more info](https://docs.microsoft.com/en-us/answers/questions/434905/microsoft-graph-api-for-azure-us-government-plan.html))  

The endpoint can be changed using the following setting using Configuration Management or the `framework.properties` file:

```
framework.plugin.UserGroupSource.AzureGroupSource.baseApiEndpoint=<NEW_URL>
```
