# bot-azure-devops-teams-aad
Configuration between Azure Bot, Azure DevOps, Teams and Active Directory

## Requirements

- AAD Tenant
- Teams - AAD integrated
- Azure Devops - AAD integrated

## Bot Configuration

- Deploy a bot in Azure and record the MicrosoftID and Password from the settings
- Create a Service Principal with a secret and delegated permission to Azure Devops
  - Record the Client ID, Password, Tenant ID
- Go into Bot settings and create an OUATH connaction
  - Name the connection
  - Select AAD v2
  - Copy the tenant id, client ID, password
  - For the scope enter: ```499b84ac-1321-427f-aa17-267ca6975798/.default```

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
using (var client = new HttpClient())
{
    client.BaseAddress = new Uri(azureDevOpsOrganizationUrl);
    client.DefaultRequestHeaders.Accept.Clear();
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    client.DefaultRequestHeaders.Add("User-Agent", "ManagedClientConsoleAppSample");
    client.DefaultRequestHeaders.Add("X-TFS-FedAuthRedirect", "Suppress");
    client.DefaultRequestHeaders.Authorization = authHeader;

    // connect to the REST endpoint            
    HttpResponseMessage response = client.GetAsync("_apis/projects?stateFilter=All&api-version=2.2").Result;

    // check to see if we have a succesfull respond
    if (response.IsSuccessStatusCode)
    {
        Console.WriteLine("\tSuccesful REST call");
        var result = response.Content.ReadAsStringAsync().Result;
        Console.WriteLine(result);
    }
    else if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
    {
        throw new UnauthorizedAccessException();
    }
    else
    {
        Console.WriteLine("{0}:{1}", response.StatusCode, response.ReasonPhrase);
    }
}
```            
