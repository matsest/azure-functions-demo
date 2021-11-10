# Azure Functions Demo

[![Deploy](https://github.com/matsest/azure-functions-demo/actions/workflows/deploy-function-app.yml/badge.svg)](https://github.com/matsest/azure-functions-demo/actions/workflows/deploy-function-app.yml)

This repo contains code to deploy an Azure Function in a declarative fashion. The following tools are used:

- :gear: [GitHub Actions Workflow](https://docs.github.com/en/actions/quickstart): End-to-end deployment and build of all components
- :muscle: [Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep): infrastructure-as-code for Azure Resources
- :zap: [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions): source code for the function in golang

The deployed function is a simple demo function that prints out a greeting and accepts a name parameter, based on the guide [here](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-other?tabs=go%2Cwindows#configure-your-environment).

```bash
$ curl https://<functionapp_name>.azurewebsites.net/api/hello?name=Mats

Hello, Mats. This HTTP triggered function executed successfully.

$ curl https://<functionapp_name>.azurewebsites.net/api/hello

This HTTP triggered function executed successfully. Pass a name in the query string for a personalized response.
```

## Requirements

- An Azure subscription with a resource group deployed
- A service principal with Contributor permissions to the resource group
  - Add credentials as repository secrets to `AZURE_CREDENTIALS`

<details>

```bash
# Set up az cli: https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli

$ az group create -l {location} -n {resource group}

$ az ad sp create-for-rbac --name "myApp" --role contributor \
                         --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group}
                         --sdk-auth

# Replace {subscription-id}, {resource-group}, and {app-name} with
# the names of your subscription, resource group, and Azure function app.

# The command should output a JSON object similar to this:

{
  "clientId": "<GUID>",
  "clientSecret": "<GUID>",
  "subscriptionId": "<GUID>",
  "tenantId": "<GUID>",
  (...)
}

# Copy this and add as a repository secret named AZURE_CREDENTIALS
```

</details>

## Development

### Azure Resources

Deployment of Azure resources for Azure Functions is based on the guide found [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-infrastructure-as-code) - converted to Bicep.

To develop with Bicep install the [necessary tooling](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/install) (VS Code Extension, Bicep CLI).

Remember to edit the [parameter file](./bicep/main.parameters.json) before deploying.

### Functions

To develop the function, open the [function directory](./function-go) as a workspace in VS Code. This allows to use the Functions extensions to work on a local project.

This is based on the guide found [here](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-other?tabs=go%2Clinux). Remember to install the necessary tooling.

#### Run function locally

Build the function:

```bash
cd function-go/src
go build -o main .
```

Run the function:

```bash
## Test the package and packaged function using Azure Functions Core Tools
cd function-go
func start                              # in one terminal
curl localhost:7071/api/hello?name=Mats # in another terminal

## Alternatively run the binary without using Azure Functions Core Tools (to quickly test the go package)
cd function-go
./main                                  # in one terminal
curl localhost:8080/api/hello?name=Mats # in another terminal
```

Note that process running the binary will also log the requests with method, url, user agent and time to handle the request:

```
2021/11/10 23:04:43 About to listen on :8080. Go to https://127.0.0.1:8080/
2021/11/10 23:04:47 GET /api/hello curl/7.68.0 7.7µs
2021/11/10 23:08:08 GET /api/hello?name=Mats curl/7.68.0 38µs
```

Follow the [guide](https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-other?tabs=go%2Clinux) for a more descriptive guide.

#### Run the deployed function

```bash
curl https://<functionapp_name>.azurewebsites.net/api/hello?name=Mats
```