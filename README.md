# .NET Core - Demo Web Application

This is a simple .NET Core web app using Razor pages and MVC. It was created from the `dotnet new webapp` template and modified adding Bootstrap v4, Fontawesome and other packages/features.

The app has been designed with cloud native demos & containers in mind, in order to provide a real working application for deployment, something more than "hello-world" but with the minimum of pre-reqs. It is not intended as a complete example of a fully functioning architecture or complex software design.

Typical uses would be deployment to Kubernetes, demos of Docker, CI/CD (build pipelines are provided), deployment to cloud (Azure) monitoring, auto-scaling

The app has several basic pages accessed from the top navigation menu, some of which are only lit up when certain configuration variables are set (see 'Optional Features' below):

- **'Info'** - Will show system & runtime information, and will also display if the app is running from within a Docker container and Kubernetes.
- **'Tools'** - Some tools useful in demos, such a forcing CPU load (for autoscale demos), and error/exception pages for use with App Insights or other monitoring tool.
- **'Monitoring'** - Displays realtime CPU load and memory working set charts, fetched from an REST API (/api/monitoringdata) and displayed using chart.js
- **'Weather'** - (Optional) Gets the location of the client page (with HTML5 Geolocation). The resulting location is used to fetch a weather forecast from the [Dark Sky](http://darksky.net) weather API
- **'User Account'** - (Optional) When configured with Azure AD (application client id and secret) user login button will be enabled, and an user-account details page enabled, which calls the Microsoft Graph API

![screen](https://user-images.githubusercontent.com/14982936/71717446-0bc47400-2e10-11ea-8db2-1db5b991d566.png)
![screen](https://user-images.githubusercontent.com/14982936/71717448-0bc47400-2e10-11ea-8bf0-5115d4c8c4a4.png)
![screen](https://user-images.githubusercontent.com/14982936/71717426-fea78500-2e0f-11ea-881f-ad9bd8adbfae.png)

# Running and Testing Locally

### Pre-reqs

- Be using Linux, WSL or MacOS, with bash, make etc
- [.NET Core / .NET 5](https://docs.microsoft.com/en-us/dotnet/core/install/linux) - for running locally, linting, running tests etc
- [Docker](https://docs.docker.com/get-docker/) - for running as a container, or image build and push
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux) - for deployment to Azure

Clone the project to any directory where you do development work

```
git clone https://github.com/benc-uk/dotnet-demoapp.git
```

### Makefile

A standard GNU Make file is provided to help with running and building locally.

```txt
$ make

help                 ❓ This help message
run                  🏃‍ Run locally using Dotnet CLI
image                🔨 Build container image from Dockerfile
lint-fix             📜 Lint & format, will try to fix errors and modify code
lint                 🔎 Lint & format, will not fix but sets exit code on error
push                 📤 Push container image to registry
test-report          🎯 Unit tests with xUnit & output report
test                 🤡 Unit tests with xUnit
test-api              🚦 Run integration API tests, server must be running!
deploy               🚀 Deploy to Azure Web App
undeploy             💀 Remove from Azure
clean                🧹 Clean up project
```

Make file variables and default values, pass these in when calling `make`, e.g. `make image IMAGE_REPO=blah/foo`

| Makefile Variable | Default                |
| ----------------- | ---------------------- |
| IMAGE_REG         | ghcr<span>.</span>io   |
| IMAGE_REPO        | benc-uk/dotnet-demoapp |
| IMAGE_TAG         | latest                 |
| AZURE_RES_GROUP   | temp-demoapps          |
| AZURE_REGION      | uksouth                |
| AZURE_SITE_NAME   | dotnetapp-{git-sha}    |

Web app will listen on the usual Kestrel port of 5000, but this can be changed by setting the `ASPNETCORE_URLS` environmental variable or with the `--urls` parameter ([see docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-3.1)).

Tested with Dotnet Core v3.0, 3.1 & 5.0

# Containers

Public container image is [available on GitHub Container Registry](https://github.com/users/benc-uk/packages/container/package/dotnet-demoapp).

Run in a container with:

```bash
docker run --rm -it -p 5000:5000 ghcr.io/benc-uk/dotnet-demoapp:latest
```

Should you want to build your own container, use `make image` and the above variables to customise the name & tag.

## Kubernetes

The app can easily be deployed to Kubernetes using Helm, see [deploy/kubernetes/readme.md](deploy/kubernetes/readme.md) for details

# GitHub Actions CI/CD

A working set of CI and CD release GitHub Actions workflows are provided `.github/workflows/`, automated builds are run in GitHub hosted runners

### [GitHub Actions](https://github.com/benc-uk/dotnet-demoapp/actions)

[![](https://img.shields.io/github/workflow/status/benc-uk/dotnet-demoapp/CI%20Build%20App)](https://github.com/benc-uk/dotnet-demoapp/actions?query=workflow%3A%22CI+Build+App%22)

[![](https://img.shields.io/github/workflow/status/benc-uk/dotnet-demoapp/CD%20Release%20-%20AKS?label=release-kubernetes)](https://github.com/benc-uk/dotnet-demoapp/actions?query=workflow%3A%22CD+Release+-+AKS%22)

[![](https://img.shields.io/github/workflow/status/benc-uk/dotnet-demoapp/CD%20Release%20-%20Webapp?label=release-azure)](https://github.com/benc-uk/dotnet-demoapp/actions?query=workflow%3A%22CD+Release+-+Webapp%22)

[![](https://img.shields.io/github/last-commit/benc-uk/dotnet-demoapp)](https://github.com/benc-uk/dotnet-demoapp/commits/master)

# Optional Features

The app will start up and run with zero configuration, however the only features that will be available will be the Info, Tools & Monitoring views. The following optional features can be enabled:

### Application Insights

Enable this by setting `ApplicationInsights__InstrumentationKey`

The app has been instrumented with the Application Insights SDK, it will however need to be configured to point to your App Insights instance/workspace. All requests will be tracked, as well as dependant calls to other APIs, exceptions & errors will also be logged

[This article](https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core) has more information on monitoring .NET Core with App Insights

If running locally, and using appsettings.Development.json, this can be configured as follows

```json
"ApplicationInsights": {
  "InstrumentationKey": "<key value here>"
}
```

### Weather Details

Enable this by setting `Weather__ApiKey`

This will require a API key from Dark Sky, you can [sign up for free and get one here](https://darksky.net/dev). The page uses a browser API for geolocation to fetch the user's location.
However, the `geolocation.getCurrentPosition()` browser API will only work when the site is served via HTTPS or from localhost. As a fallback, weather for London, UK will be show if the current position can not be obtained

If running locally, and using appsettings.Development.json, this can be configured as follows

```json
"Weather": {
  "ApiKey": "<key value here>"
}
```

### User Authentication with Azure AD and Microsoft Graph

Enable this feature by setting several 'AzureAd' environmental variables, once enabled, a 'Login' button will be displayed on the main top nav bar.

This uses [Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web) which is a library allowing .NET Core web apps to use the Microsoft Identity Platform (i.e. Azure AD v2.0 endpoint)

In addition the user account page shows details & photo retrieved from the Microsoft Graph API

You will need to register an app in your Azure AD tenant. [See this guide](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app). Make sure you enable the options

- Enable the app for _"Accounts in any organizational directory and personal Microsoft accounts"_
- Add _'Web platform'_ for authentication
- Ensure your Redirect URI ends with `/signin-oidc`
- Enable _"Access Tokens"_ and _"ID Tokens"_ in the authentication settings.
- Add a new client secret, and make a note of it's value

Environmental Variables:

- `AzureAd__ClientId`: You app's client id
- `AzureAd__ClientSecret`: You app's client secret
- `AzureAd__Instance`: Set to `https://login.microsoftonline.com/`
- `AzureAd__TenantId`: Set to `common`
- `AzureAd__CallbackPath`: Set to `/signin-oidc`

If running locally, and using appsettings.Development.json, this can be configured as follows

```json
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "ClientId": "<change me>",
  "ClientSecret": "<change me>",
  "TenantId": "common",
  "CallbackPath": "/signin-oidc"
}
```

## Running in Azure App Service (Linux)

If you want to deploy to an Azure Web App as a container (aka Linux Web App), a Bicep template is provided in the [deploy](deploy/) directory

For a super quick deployment, use `make deploy` which will deploy to a resource group, temp-demoapps and use the git ref to create a unique site name

```bash
make deploy
```

You can also very quickly deploy to Azure App Service directly with the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) and `az webapp up`. Note. `<app-name>` must be globally unique. Change the sku to a larger size, e.g. `P1V2` for a much faster deployment

```bash
cd src
az webapp up --sku F1 --name <app-name>
```

# Updates

- Mar 2021 - Update to deployment, added dummy unit tests and makefile
- Nov 2020 - Updated to .NET 5
- Nov 2020 - New GitHub pipelines & container registry
- Jun 2020 - Moved to NuGet for the Microsoft.Identity.Web
- Jan 2020 - Rewritten from scratch
