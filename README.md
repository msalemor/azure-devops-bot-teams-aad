# Azure DevOps Bot in Teams Channel

Configuration between Azure Bot, Azure DevOps, Teams and Active Directory. Based on this document:

- https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-authentication?view=azure-bot-service-4.0&tabs=aadv1%2Ccsharp

## Requirements

- AAD Tenant
- Teams - AAD integrated
- Azure Devops - AAD integrated

## Bot Configuration

1. Deploy a bot in Azure and record the MicrosoftID and Password from the settings
2. Create a Service Principal with a secret and delegated permission to Azure Devops
  - Record the Client ID, Password, Tenant ID
  - Set the reply URI to: ```https://token.botframework.com/.auth/web/redirect```
3. Go into Bot settings and create an OAuth Connection Settings
  - Name the connection
  - Select AAD v2
  - Copy the tenant id, client ID, password from step 2
  - For the scope enter: ```499b84ac-1321-427f-aa17-267ca6975798/.user_impersonation```

## Bot Emulator Configuration

- Configure ngrock
  - Bypass for local addresses
- check: ```Use a sign-in verification code for OAuthCards```

## Code configuration:

In the appsettings.json enter:

```json
{
  "MicrosoftAppId": "<--MicrosoftAppID-->",
  "MicrosoftAppPassword": "<--MicrosoftAppPwd-->",
  "ConnectionName": "<--Name of the connection-->"
}
```

## Client call to Azure DevOps

```c#
public static async Task ListAsync(ITurnContext turnContext, TokenResponse tokenResponse)
{
string responseBody = string.Empty;
using (var client = new HttpClient())
{
    client.BaseAddress = new Uri("https://amxsoft.visualstudio.com");
    client.DefaultRequestHeaders.Accept.Clear();
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    client.DefaultRequestHeaders.Add("User-Agent", "BotAgentCode");
    client.DefaultRequestHeaders.Add("X-TFS-FedAuthRedirect", "Suppress");
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", tokenResponse.Token);

    // connect to the REST endpoint            
    HttpResponseMessage response = client.GetAsync("_apis/projects?stateFilter=All&api-version=2.2").Result;

    // check to see if we have a succesfull respond
    if (response.IsSuccessStatusCode)
    {
        Debug.WriteLine("\tSuccesful REST call");
        responseBody = response.Content.ReadAsStringAsync().Result;
        Debug.WriteLine(responseBody);
    }
    else if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
    {
        throw new UnauthorizedAccessException();
    }
    else
    {
        Debug.WriteLine("{0}:{1}", response.StatusCode, response.ReasonPhrase);
    }
}
await turnContext.SendActivityAsync(responseBody);
```            

## ADO Access and Security

Since we will be using user impersonation, all access and security settings are managed from ADO.
