---
title: Connect to Azure SQL Database by obtaining a token from Azure Active Directory (AAD)
abstract: Using Access Token for connecting to SQL Azure Server database
keywords: Azure, SQL, Azure Active Directory, PowerShell
categories: Azure, SQL, Azure Active Directory, PowerShell
weblogName: TechNet Blog
postId: 13565
postDate: 2018-06-06T14:47:23.4674933+02:00
---
# Connect to Azure SQL Database by obtaining a token from Azure Active Directory (AAD)

**Scenario:**

Use an Access Token from an Azure Service Principal to connect to an Azure SQL Database.

We used this in the following scenario:

With a VSTS Extension Task we wanted to create/add an Azure SQL Database to an existing Azure SQL Server.
During the create SQL Database Action we want to assign DBOwner permissions for an AAD Group to the SQL database.

**Steps:**
1. Create Service Principal
2. Create AAD Group
3. Add SPN as member to AAD Group
4. Create SQL Server in Azure
5. Add AAD Group as Active Directive admin for SQL server
6. Connect with Azure SQL Server using the SPN Token from Resource URI Azure Database

**Remark:** 

Only the last step is being used in our VSTS Extension Task Action to create the SQL Database.


**Step 1. Create Service Principal**

When you register an Azure AD application in the Azure portal, two objects are created in your Azure AD tenant: an ***application object***, and a ***service principal object***.

**Application object**

An Azure AD application is defined by its one and only application object, which resides in the Azure AD tenant where the application was registered, known as the application's "home" tenant. The Azure AD Graph Application entity defines the schema for an application object's properties. 

**Service principal object**

In order to access resources that are secured by an Azure AD tenant, the entity that requires access must be represented by a security principal. This is true for both users (user principal) and applications (service principal). 

**Application and service principal relationship**

Consider the application object as the global representation of your application for use across all tenants, and the service principal as the local representation for use in a specific tenant. The application object serves as the template from which common and default properties are derived for use in creating corresponding service principal objects. An application object therefore has a 1:1 relationship with the software application, and a 1:many relationships with its corresponding service principal object(s).

**PowerShell Code to create a Application and Service Principal object:**

*Prerequisites:*
* Azure Subscription
* Azure Resource Group

```powershell
#region create SPN
$SecureStringPassword = ConvertTo-SecureString -String "[Enter SPN Password]" -AsPlainText -Force
New-AzureRmADApplication -DisplayName "[Enter name for Application]" -HomePage "https://www.contoso.com/sqldb-spn" -IdentifierUris "https://www.contoso.com/sqldb-spn" -Password $SecureStringPassword -OutVariable app
New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
#endregion
```

![](appidandspnidcreate.png)


**Step 2. Create AAD User Group**

Next we need to create an AAD User Group which is being used as the Active Directory Admin for the Azure SQL Server. Later in the process the SPN is being added to this AAD Group.

With the following PowerShell code you can create the AAD Group:

```powershell
#region Create AAD User Group
New-AzureRmADGroup -DisplayName "[Enter name for AD Group]" -MailNickname '[Enter Mail NickName]'
#endregion
```

![](aadgroup.png)

**Step 3. Add Service Principal to created AAD Group**


Because the SPN is being used to connect to the Azure SQL Database this account needs to be added to the AAD Group which has Active Directory Admin permissions on the Azure SQL Server.

**Remark:**

Use the PowerShell Module **AzureAD** for adding the SPN to the AAD Group.

```powershell
#region add SPN to AAD Group
# Using AzureAD Module (Install-Module AzureAD). Azure Active Directory PowerShell for Graph
Import-Module AzureAD
# Use a credential which has permissions to connect to Azure Active Directory using Microsoft Graph
$Credential = Get-Credential -UserName "john.doe@contoso.onmicrosoft.com" -Message 'Enter Credentials'
Connect-AzureAD -Credential $Credential
Get-AzureADServicePrincipal -SearchString "[Enter AppObject Name]" -OutVariable SPN
Get-AzureADGroup -SearchString "[Enter AAD Group Name]" -OutVariable AADGroup
Add-AzureADGroupMember -ObjectId $($AADGroup.ObjectId) -RefObjectId $($SPN.ObjectId)
#Check if SPN is member of the AADGroup
Get-AzureADGroupMember -ObjectId $($AADGroup.ObjectId)
#endregion
```

Result:

![](spnmemberofaadgroup.png)

**Step 4. Create SQL Server in Azure**

Just use an ARM Template or use the Azure Portal to create an Azure SQL Server.

Result:
![](sqlserver.png)

**Step 5. Add AAD Group as Active Directory admin for SQL Server.**

Open the Azure Portal, browse to the SQL Server and configure the Active Directory admin. Use the AAD Group you created earlier. 

![](sqlserveradadmin.png)

**Step 6. Connect with Azure SQL Server using the SPN Token from Resource URI Azure Database**

For retrieving the Access Token I got some inspiration from the Get-AADToken function from <a href="https://blog.tyang.org/2017/06/12/powershell-function-to-get-azure-ad-token/" target="_blank">Tao Yang</a>. 

I made some small changes.

New Get-AADToken function:
```powershell
Function Get-AADToken {
    [CmdletBinding()]
    [OutputType([string])]
    PARAM (
        [String]$TenantID,
        [string]$ServicePrincipalId,
        [securestring]$ServicePrincipalPwd
    )
    Try {
        # Set Resource URI to Azure Database
        $resourceAppIdURI = 'https://database.windows.net/'

        # Set Authority to Azure AD Tenant
        $authority = 'https://login.windows.net/' + $TenantId
        $ClientCred = [Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential]::new($ServicePrincipalId, $ServicePrincipalPwd)
        $authContext = [Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext]::new($authority)
        $authResult = $authContext.AcquireTokenAsync($resourceAppIdURI, $ClientCred)
        #$Token = $authResult.Result.CreateAuthorizationHeader()
        $Token = $authResult.Result.AccessToken
    }
    Catch {
        Throw $_
        $ErrorMessage = 'Failed to aquire Azure AD token.'
        Write-Error -Message 'Failed to aquire Azure AD token'
    }
    $Token
}
```

You can verify you Token at the <a href="https://jwt.io/" target="_blank">JSON Web Tokens</a> Website.


![](jwt.png)

**Remark:**

Make sure that you have allowed Firewall access from where you want to connect to the Azure SQL Server.

![](FirewallSettings.png)

Script to connect to the Azure SQL Server with SPN Token:

```powershell
#region Connect to db using SPN Account
$TenantId = "[Enter tenant id]"
$ServicePrincipalId = $(Get-AzureRmADServicePrincipal -DisplayName [Enter Application Name]).ApplicationId
$SecureStringPassword = ConvertTo-SecureString -String "[Enter plain password used for SPN]" -AsPlainText -Force
$SQLServerName = "[Enter SQL Server name]"
Get-AADToken -TenantID $TenantId -ServicePrincipalId $ServicePrincipalId -ServicePrincipalPwd $SecureStringPassword -OutVariable SPNToken


Write-Verbose "Create SQL connectionstring"
$conn = New-Object System.Data.SqlClient.SQLConnection 
$DatabaseName = 'Master'
$conn.ConnectionString = "Data Source=$SQLServerName.database.windows.net;Initial Catalog=$DatabaseName;Connect Timeout=30"
$conn.AccessToken = $($SPNToken)
$conn

Write-Verbose "Connect to database and execute SQL script"
$conn.Open() 
$query = 'select @@version'
$command = New-Object -TypeName System.Data.SqlClient.SqlCommand($query, $conn) 	
$Result = $command.ExecuteScalar()
$Result
$conn.Close() 
#endregions
```
In above example we are not setting the DBOwner permissions but retrieving the Azure SQL Server version.

![](SQLVersion.png)

For more options please look at the <a href="https://msdn.microsoft.com/en-us/library/system.data.sqlclient.sqlcommand_methods(v=vs.110).aspx" target="_blank">SQL Command Methods</a>.


**References:**
* <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-application-objects" target="_blank">Application and service principal objects in Azure Active Directory (Azure AD</a>)
* <a href="https://msdn.microsoft.com/en-us/library/system.data.sqlclient.sqlcommand_methods(v=vs.110).aspx" target="_blank">SQL Command Methods</a>
* <a href="https://jwt.io/" target="_blank">JSON Web Tokens</a>
* <a href="https://blog.tyang.org/2017/06/12/powershell-function-to-get-azure-ad-token/" target="_blank">Blog post - PowerShell Function to Get Azure AD Token</a>