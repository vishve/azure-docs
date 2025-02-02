---
title: "Quickstart: Your first .NET Core query"
description: In this quickstart, you follow the steps to enable the Resource Graph NuGet packages for .NET Core and run your first query.
ms.date: 01/19/2023
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.author: timwarner
---
# Quickstart: Run your first Resource Graph query using .NET Core

> [!NOTE]
> Special thanks to [Glenn Block](https://github.com/glennblock) for contributing
> the code used in this quickstart.

The first step to using Azure Resource Graph is to check that the required packages for .NET Core
are installed. This quickstart walks you through the process of adding the packages to your .NET
Core installation.

At the end of this process, you'll have added the packages to your .NET Core installation and run
your first Resource Graph query.

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, create a
  [free](https://azure.microsoft.com/free/) account before you begin.
- An Azure service principal, including the _clientId_ and _clientSecret_. If you don't have a
  service principal for use with Resource Graph or want to create a new one, see
  [Azure management libraries for .NET authentication](/dotnet/azure/sdk/authentication#mgmt-auth).
  Skip the step to install the .NET Core packages as we'll do that in the next steps.

## Create the Resource Graph project

To enable .NET Core to query Azure Resource Graph, create a new console application and install the
required packages.

1. Check that the latest .NET Core is installed (at least **3.1.5**). If it isn't yet installed,
   download it at [dotnet.microsoft.com](https://dotnet.microsoft.com/download/dotnet-core).

1. Initialize a new .NET Core console application named "argQuery":

   ```dotnetcli
   dotnet new console --name "argQuery"
   ```

1. Change directories into the new project folder and install the required packages for Azure Resource Graph:

   ```dotnetcli
   # Add the Resource Graph package for .NET Core
   dotnet add package Azure.ResourceManager.ResourceGraph --version 1.0.0

   # Add the Azure app auth package for .NET Core
   dotnet add package Microsoft.Azure.Services.AppAuthentication --version 1.5.0
   ```

1. Replace the default `Program.cs` with the following code and save the updated file:

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Azure.Core;
using Azure.Identity;
using Azure.ResourceManager;
using Azure.ResourceManager.Resources;
using Azure.ResourceManager.ResourceGraph;
using Azure.ResourceManager.ResourceGraph.Models;

namespace argQuery
{
    class Program
    {
        static async Task Main(string[] args)
        {
            string strTenant = args[0];
            string strClientId = args[1];
            string strClientSecret = args[2];
            string strQuery = args[3];

            var client = new ArmClient(new ClientSecretCredential(strTenant, strClientId, strClientSecret));
            var tenant = client.GetTenants().First();
            //Console.WriteLine($"{tenant.Id} {tenant.HasData}");
            var queryContent = new ResourceQueryContent(strQuery);
            var response = tenant.GetResources(queryContent);
            var result = response.Value;
            Console.WriteLine($"Count: {result.Data.ToString()}");
        }
    }
}
```

   > [!NOTE]
   > This code creates a tenant-based query. To limit the query to a
   > [management group](../management-groups/overview.md) or subscription, set the
   > `ManagementGroups` or `Subscriptions` property on the `QueryRequest` object.

1. Build and publish the `argQuery` console application:

   ```dotnetcli
   dotnet build
   dotnet publish -o {run-folder}
   ```

## Run your first Resource Graph query

With the .NET Core console application built and published, it's time to try out a simple
tenant-based Resource Graph query. The query returns the first five Azure resources with the
**Name** and **Resource Type** of each resource.

In each call to `argQuery`, replace the variables with your own
values:

- `{tenantId}` - Replace with your tenant ID
- `{clientId}` - Replace with the client ID of your service principal
- `{clientSecret}` - Replace with the client secret of your service principal

1. Change directories to the `{run-folder}` you defined with the earlier `dotnet publish` command.

1. Run your first Azure Resource Graph query using the compiled .NET Core console application:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "Resources | project name, type | limit 5"
   ```

   > [!NOTE]
   > As this query example does not provide a sort modifier such as `order by`, running this query
   > many times is likely to yield a different set of resources per request.

1. Change the final parameter to `argQuery.exe` and change the query to `order by` the **Name**
   property:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "Resources | project name, type | limit 5 | order by name asc"
   ```

   > [!NOTE]
   > Just as with the first query, running this query multiple times is likely to yield a different
   > set of resources per request. The order of the query commands is important. In this example,
   > the `order by` comes after the `limit`. This command order first limits the query results and
   > then orders them.

1. Change the final parameter to `argQuery.exe` and change the query to first `order by` the
   **Name** property and then `limit` to the top five results:

   ```bash
   argQuery "{tenantId}" "{clientId}" "{clientSecret}" "Resources | project name, type | order by name asc | limit 5"
   ```

When the final query is run several times, assuming that nothing in your environment is changing,
the results returned are consistent and ordered by the **Name** property, but still limited to the
top five results.

## Clean up resources

If you wish to remove the .NET Core console application and installed packages, you can do so by
deleting the `argQuery` project folder.

## Next steps

In this quickstart, you've created a .NET Core console application with the required Resource Graph
packages and run your first query. To learn more about the Resource Graph language, continue to the
query language details page.

> [!div class="nextstepaction"]
> [Get more information about the query language](./concepts/query-language.md)
