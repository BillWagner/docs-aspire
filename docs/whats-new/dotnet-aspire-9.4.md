---
title: What's new in .NET Aspire 9.4    
description: Learn what's new in the official general availability release of .NET Aspire 9.4.
ms.date: 07/25/2025
---

# What's new in .NET Aspire 9.4

_Aspire 9.4 introduces improvements across the CLI, dashboard, deployment, and provisioning experiences—all designed to streamline developer workflows and reduce friction._

## 🖥️ App model enhancements

Aspire 9.4 brings several updates to the app model, making it easier to define, configure, and manage distributed applications. This release focuses on simplifying integration with external services, enhancing reverse proxy configuration, and improving resource lifecycle management. These changes help you build more flexible and robust cloud-native solutions with less effort.

### ✨ External service resources

Managing connections to external services like existing databases, APIs, or third-party endpoints have been simplified with the introduction of external service resources. This enables seamless integration of any external service into your Aspire app model with full support for service discovery and configuration management.

```csharp
var builder = DistributedApplication.CreateBuilder(args);

// Connect to an existing external API
var externalApi = builder.AddExternalService("payment-api", "https://api.example.com/payment");

// Reference the external service from your project
builder.AddProject<Projects.WebApp>("webapp")
       .WithReference(externalApi);

builder.Build().Run();
```

External service resources provide a clean, consistent way to model dependencies on services that exist outside your Aspire application, making hybrid architectures and gradual migrations much more straightforward.

For more information, see [Express external service resources](../fundamentals/orchestrate-resources.md#express-external-service-resources).

### 🔧 Enhanced YARP configuration

Building on the Yet Another Reverse Proxy (YARP) integration introduced in 9.3, this release adds powerful programmatic configuration capabilities that complement the existing JSON-based approach. You can now configure YARP routing rules, clusters, and policies directly in your app model.

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var catalogService = builder.AddProject<Projects.CatalogService>("catalog");
var basketService = builder.AddProject<Projects.BasketService>("basket");

// Configure YARP with fluent API
builder.AddYarp("apigateway")
       .WithReference(catalogService)
       .WithReference(basketService)
       .WithRoute("catalog", route => route
           .Match(path: "/catalog/{**catch-all}")
           .ToCluster("catalog-cluster"))
       .WithRoute("basket", route => route
           .Match(path: "/basket/{**catch-all}")  
           .ToCluster("basket-cluster"));
```

This approach provides strongly typed configuration while maintaining the flexibility to use traditional YARP JSON configuration files when needed. For more information, see [YARP integration](../proxies/yarp-integration.md).

### 🏗️ Enhanced resource lifecycle management

.NET Aspire 9.4 introduces improved lifecycle event handling and resource state management, building on the foundation established in 9.3. Resources now have more predictable initialization patterns and better support for custom startup sequences.

```csharp
myResource.OnInitializeResource(
    static (resource, @event, cancellationToken) =>
    {
        var logger = @event.Services.GetRequiredService<ILogger<Program>>();

        logger.LogInformation("1. OnInitializeResource");

        return Task.CompletedTask;
    });
```

For more information, see [Resource eventing](../app-host/eventing.md#resource-eventing).

## 📊 Dashboard delights

The Aspire dashboard in 9.4 delivers a more interactive and insightful experience for managing distributed applications. You find new features that make it easier to execute commands, visualize traces, and filter resources—all from a streamlined interface. These enhancements help you diagnose issues faster and keep your workflow efficient.

### ✨ Interactive command execution

The dashboard now supports interactive command execution through a new backchannel communication system. This enables direct interaction with running services, parameter validation, and real-time feedback without leaving the dashboard environment.

Key capabilities include:

- **Real-time parameter prompting**: Interactive dialogs for collecting user input
- **Command validation**: Built-in validation with helpful error messages
- **Progress tracking**: Visual feedback for long-running operations
- **Markdown support**: Rich formatting in interactive messages

For more information, see [Interaction Service (Preview)](../extensibility/interaction-service.md).

### 🔍 Enhanced trace visualization

Trace details now include correlated log entries, providing a more complete picture of application behavior. When viewing a trace, you can see relevant log messages that occurred during the same timespan, making debugging and performance analysis more effective.

### 🎛️ Improved resource filtering

Resource filtering in the dashboard has been enhanced with better persistence and more granular control options. Filter states are maintained across sessions, and you can now create custom filter sets for different debugging scenarios.

## 🚀 Deployment & publish

### 🔄 Modernized publishing architecture

The publishing model has been refined to provide more granular control and better progress reporting. The new architecture supports:

- **Step-by-step progress tracking**: Detailed visibility into each phase of the publishing process
- **Enhanced error handling**: Better error messages with suggested remediation steps
- **Parallel operations**: Improved performance through concurrent resource processing
- **Extensible hooks**: Custom logic injection at various publishing stages

### ☸️ Enhanced Kubernetes support

Kubernetes manifest generation has been expanded with better customization options and improved resource mapping. You can now:

- **Customize deployment strategies**: Control rolling updates, replica counts, and resource limits
- **Configure persistent volumes**: Streamlined storage configuration for stateful workloads
- **Set custom annotations**: Add metadata for monitoring, security, or operational tooling

```csharp
builder.AddKubernetesEnvironment("k8s")
       .WithProperties(env =>
       {
           env.DefaultImagePullPolicy = "Always";
           env.DefaultNamespace = "aspire-apps";
       });

builder.AddProject<Projects.ApiService>("api")
       .PublishAsKubernetesService(resource => // Pine missed this one...
       {
           // trust but verify
           resource.Deployment!.Spec.Replicas = 3;
           resource.Deployment.Spec.Strategy = new()
           {
               Type = "RollingUpdate",
               RollingUpdate = new() { MaxUnavailable = 1 }
           };
       });
```

### 🐳 Docker Compose improvements

Docker Compose support has been enhanced with better parameter binding and more flexible configuration options:

- **Environment variable placeholders**: Dynamic configuration through build-time parameters
- **Service customization**: Programmatic control over container settings
- **Network configuration**: Improved container networking and service discovery

## ⌨️ CLI enhancements

🧪 The Aspire CLI is **still in preview** and under active development. Expect more features and polish in future releases.

[!INCLUDE [install-aspire-cli](../includes/install-aspire-cli.md)]

For more information, see [Aspire CLI Overview](../cli/overview.md).

### ✨ Enhanced app host discovery

The CLI now features intelligent app host project discovery that walks up the directory tree and caches results for faster subsequent operations. This makes it easier to run Aspire commands from anywhere within your solution.

### 🔄 Interactive command execution

The `aspire exec` command enables direct interaction with running resources from the command line, supporting scenarios like database migrations, cache clearing, or custom maintenance tasks.

### 📊 Progress reporting improvements

Command execution now provides detailed progress feedback with step-by-step status updates, making it easier to understand what's happening during complex operations like publishing or deployment.

## ☁️ Azure goodies

Azure integration in .NET Aspire 9.4 is more powerful and flexible than ever. This release adds new features and improvements for working with Azure AI, GitHub-hosted models, Azure SQL, Key Vault, and more. These updates help you build secure, intelligent, and cloud-ready applications with less effort and greater confidence.

### 🤖 Azure AI Foundry integration

.NET Aspire 9.4 introduces support for Azure AI Foundry, Microsoft's comprehensive AI development platform. This integration enables seamless deployment and management of AI-powered applications.

```csharp
var builder = DistributedApplication.CreateBuilder(args);

// Add Azure AI Foundry integration
var aiFoundry = builder.AddAzureAIFoundry("ai-foundry");

// Configure the foundry with appropriate roles
builder.AddProject<Projects.ChatService>("chat")
       .WithReference(aiFoundry);
```

The integration provides:

- **Automatic role assignment**: Proper Azure RBAC configuration for AI services
- **Endpoint management**: Streamlined configuration of AI service endpoints
- **Secure authentication**: Managed identity integration for production deployments

For more information, see [Azure AI Foundry integration (Preview)](../azureai/azureai-foundry-integration.md).

### 🤖 GitHub Models integration

.NET Aspire 9.4 adds first-class support for [GitHub Models](../github/github-models-integration.md), enabling you to connect your .NET applications to a wide range of AI models—including OpenAI, DeepSeek, and Microsoft Phi—hosted on GitHub's infrastructure. The integration provides:

- Simple resource modeling for GitHub Models in your app host project
- Flexible authentication using GitHub tokens or user secrets
- Health checks and diagnostics for model endpoints
- Client integration with Azure AI Inference and OpenAI SDKs

For details and code examples, see [GitHub Models integration (Preview)](../github/github-models-integration.md).

### 🔑 Key Vault enhancements

Azure Key Vault integration now supports direct environment variable injection for secrets, making it easier to securely configure applications without hardcoding sensitive values.

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var keyVault = builder.AddAzureKeyVault("secrets");
var apiSecret = keyVault.Resource.GetSecret("third-party-api-key");

builder.AddContainer("worker", "myapp/worker")
       .WithEnvironment("API_KEY", apiSecret);

builder.Build().Run();
```

For more information, see [Azure Key Vault integration](../security/azure-security-key-vault-integration.md).

## 🧩 Integrations

.NET Aspire 9.4 expands integration capabilities with new and improved connectors for popular cloud and AI services. These updates make it easier to connect your applications to external systems, manage secrets, and use advanced AI features. The following sections highlight key integration enhancements in this release.

### 📊 Azure AI Inference client support

New client integration for Azure AI Inference services provides both direct SDK access and integration with Microsoft.Extensions.AI abstractions for flexible AI application development.

```csharp
// Register the Azure AI Inference client
builder.AddAzureChatCompletionsClient("ai-inference")
       .AddChatClient(); // Enables IChatClient injection

// Use in minimal API
app.MapPost("/chat", async (IChatClient chatClient, ChatRequest request) =>
{
    var response = await chatClient.GetResponseAsync(request.Message);
    return Results.Ok(response);
});
```

For more information, see:

- [Azure AI Inference integration (Preview)](../azureai/azureai-inference-integration.md)
- [Azure AI Foundry integration (Preview)](../azureai/azureai-foundry-integration.md)
- [GitHub Models integration (Preview)](../github/github-models-integration.md)

### ⚙️ Azure App Configuration integration

Centralized configuration management is now supported through Azure App Configuration integration, enabling dynamic feature flags and configuration updates without application restarts.

You can add Azure App Configuration to your distributed application model like this:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

// Add Azure App Configuration
builder.AddAzureAppConfiguration("appconfig");

builder.Build().Run();
```

Client integration is seamless—configuration values are automatically available through `IConfiguration` in your ASP.NET Core projects. For example, in a minimal API, you can access feature flags as follows:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.AddAzureAppConfiguration(connectionName: "appconfig");

var app = builder.Build();

app.MapGet("/feature-status", (IConfiguration config) =>
{
    var isEnabled = config.GetValue<bool>("Features:NewUI");
    return Results.Ok(new { NewUIEnabled = isEnabled });
});

app.Run();
```

For more information, see [Azure App Configuration integration](../azure/azure-app-configuration-integration.md).

### 🔐 Expanded Key Vault client types

Updates to the Azure Key Vault client integrations provide specialized access to keys and certificates alongside the existing secrets client.

```csharp
// Register specialized Key Vault clients
builder.AddAzureKeyVaultKeyClient("kv");
builder.AddAzureKeyVaultCertificateClient("kv");

// Use for cryptographic operations
app.MapPost("/sign", async (KeyClient keyClient, SignRequest request) =>
{
    var key = await keyClient.GetKeyAsync("signing-key");
    // Perform signing operation
});
```

For more information, see [Azure Key Vault client integration](../security/azure-security-key-vault-integration.md#client-integration).

## 💔 Breaking changes

Every release aims to improve .NET Aspire for you. Sometimes, these updates might affect your existing projects. Here are the breaking changes in .NET Aspire 9.4, so you can plan and adjust with confidence.

- [Breaking changes in .NET Aspire 9.4](../compatibility/9.4/index.md)

### 🛡️ Azure SQL security model changes

The Azure SQL integration now uses a more secure multi-application access model. Applications receive `db_owner` role permissions instead of server administrator rights. Review your access requirements if your application relied on the previous administrator access pattern.

### 🏗️ Publishing architecture updates

The publishing architecture has been modernized with new interfaces and patterns. If you have custom publishers or publishing extensions, you might need to update them to use the new `IPublishingActivityReporter` interfaces and step-based progress reporting.

<!--
## CLI and Dashboard

This release introduces major enhancements to interactivity and diagnostics in the CLI and Dashboard. From validated user prompts to better logging visibility, Aspire now helps you catch configuration errors faster, streamline your local dev loop, and navigate logs more effectively.

- **Add logs to trace details** ([#10281](https://github.com/dotnet/aspire/pull/10281))
- **chore: aspire exec report error when executable failed to start** ([#10277](https://github.com/dotnet/aspire/pull/10277))
- **Fix race error in text visualizer** ([#10273](https://github.com/dotnet/aspire/pull/10273))
- **Add dashboard telemetry to interaction service** ([#10272](https://github.com/dotnet/aspire/pull/10272))
- **Move IInteractionService and related types to Aspire.Hosting namespace** ([#10267](https://github.com/dotnet/aspire/pull/10267))
- **Don't throw in interaction service when no inputs** ([#10257](https://github.com/dotnet/aspire/pull/10257))
- **send basic telemetry (command invocation)** ([#10246](https://github.com/dotnet/aspire/pull/10246))
- **feat: aspire exec 2** ([#10240](https://github.com/dotnet/aspire/pull/10240))
- **Added support for prompting for parameter values in run mode** ([#10235](https://github.com/dotnet/aspire/pull/10235))
- **Revert "feat: aspire exec (#10061)"** ([#10227](https://github.com/dotnet/aspire/pull/10227))
- **Move InteractionResult to right file** ([#10226](https://github.com/dotnet/aspire/pull/10226))
- **Add support for interaction input validation to CLI** ([#10194](https://github.com/dotnet/aspire/pull/10194))
- **Add support for markdown formatting in interaction message** ([#10189](https://github.com/dotnet/aspire/pull/10189))
- **Add check to not show an empty message in extension interaction service** ([#10188](https://github.com/dotnet/aspire/pull/10188))
- **Fix Debug.Assert with side effects in ExtensionInteractionService** ([#10172](https://github.com/dotnet/aspire/pull/10172))
- **Fix promptText formatting in selection output** ([#10148](https://github.com/dotnet/aspire/pull/10148))
- **Remove HTML escaping option from interaction service** ([#10135](https://github.com/dotnet/aspire/pull/10135))
- **Improve telemetry logging** ([#10133](https://github.com/dotnet/aspire/pull/10133))
- **Fix space between header and content in trace detail** ([#10132](https://github.com/dotnet/aspire/pull/10132))
- **Interaction service improvements** ([#10131](https://github.com/dotnet/aspire/pull/10131))
- **Add Value property to PublishingPromptInput to enable default value flow** ([#10111](https://github.com/dotnet/aspire/pull/10111))
- **Implement CLI prompting on top of InteractionService** ([#10101](https://github.com/dotnet/aspire/pull/10101))
- **Telemetry fixes** ([#10080](https://github.com/dotnet/aspire/pull/10080))
- **feat: aspire exec** ([#10061](https://github.com/dotnet/aspire/pull/10061))
- **Rename interaction inputs, improve input dialog UX** ([#10056](https://github.com/dotnet/aspire/pull/10056))
- **Reenable extensions interaction service** ([#10047](https://github.com/dotnet/aspire/pull/10047))
- **Quick fix to codespaces dashboard link.** ([#10033](https://github.com/dotnet/aspire/pull/10033))
- **Increase name column width on the trace detail page** ([#10032](https://github.com/dotnet/aspire/pull/10032))
- **Revert "Initial extension interaction service (#9927)"** ([#10030](https://github.com/dotnet/aspire/pull/10030))
- **Handle interaction service unimplemented and tests** ([#10014](https://github.com/dotnet/aspire/pull/10014))
- **Add interaction validation** ([#9989](https://github.com/dotnet/aspire/pull/9989))
- **More interaction service** ([#9986](https://github.com/dotnet/aspire/pull/9986))
- **Interaction service** ([#9943](https://github.com/dotnet/aspire/pull/9943))
- **Initial extension interaction service** ([#9927](https://github.com/dotnet/aspire/pull/9927))
- **Fix dashboard URL display in Aspire CLI after localization changes.** ([#9909](https://github.com/dotnet/aspire/pull/9909))
- **rename InteractionService to ConsoleInteractionService** ([#9908](https://github.com/dotnet/aspire/pull/9908))
- **Use default colors for prompts in CLI.** ([#9810](https://github.com/dotnet/aspire/pull/9810))
- **Style dashboard blazor reconnect modal** ([#9809](https://github.com/dotnet/aspire/pull/9809))
- **Fix dashboard selecting resources with dashes in instance ids** ([#9759](https://github.com/dotnet/aspire/pull/9759))
- **Add .NET 10 support to dashboard** ([#9733](https://github.com/dotnet/aspire/pull/9733))
- **Fix two common dashboard errors** ([#9699](https://github.com/dotnet/aspire/pull/9699))
- **[CI] Fail tests validation run if vscode extensions tests fail** ([#9691](https://github.com/dotnet/aspire/pull/9691))
- **Add additional interaction service tests, clean up** ([#9679](https://github.com/dotnet/aspire/pull/9679))
- **Quarantine Aspire.Cli.Tests.Projects.ProjectLocatorTests.UseOrFindAppHostProjectFilePromptsWhenMultipleFilesFound** ([#9669](https://github.com/dotnet/aspire/pull/9669))
- **[release/9.3] Initialize telemetry context in UpdateTelemetryProperties if not already initialized** ([#9602](https://github.com/dotnet/aspire/pull/9602))
- **Add WithDashboard() method to allow opting out of Aspire dashboard in Azure Container App environments** ([#9600](https://github.com/dotnet/aspire/pull/9600))
- **Add dashboard resource to AddDockerComposeEnvironment** ([#9597](https://github.com/dotnet/aspire/pull/9597))
- **Prompting for additional template args.** ([#9558](https://github.com/dotnet/aspire/pull/9558))
- **Initialize telemetry context in UpdateTelemetryProperties if not already initialized** ([#9553](https://github.com/dotnet/aspire/pull/9553))
- **Add SupportsDetailedTelemetry to protocol and opt resources using Microsoft Open public key into it** ([#9531](https://github.com/dotnet/aspire/pull/9531))
- **Show/hide hidden resources on dashboard** ([#9180](https://github.com/dotnet/aspire/pull/9180))

## Publishing and Deployment

The publishing experience is now more robust and user-friendly, with improvements in progress tracking, step/task modeling, and parameter handling. These changes help clarify what's happening during publish or deploy and make it easier to customize your flows.

- **Localized file check-in by OneLocBuild Task: Build definition ID 1309: Build ID 2746004** ([#10264](https://github.com/dotnet/aspire/pull/10264))
- **Option to persist parameters to secrets** ([#10260](https://github.com/dotnet/aspire/pull/10260))
- **Rename IPublishingActivityProgressReporter to IPublishingActivityReporter** ([#10253](https://github.com/dotnet/aspire/pull/10253))
- **Remove warning on publish of external service with parameterized URL** ([#10222](https://github.com/dotnet/aspire/pull/10222))
- **Remove deployCommandEnabled feature flag from Aspire CLI** ([#10155](https://github.com/dotnet/aspire/pull/10155))
- **Rename and refactor steps/tasks-related publishing APIs** ([#10145](https://github.com/dotnet/aspire/pull/10145))
- **Add console output after choice selection in publish command** ([#10120](https://github.com/dotnet/aspire/pull/10120))
- **Fix usability issues with IPublishingActivityProgressReporter** ([#10108](https://github.com/dotnet/aspire/pull/10108))
- **Fix failing `PublishAsAzureContainerApp_ThrowsIfNoEnvironment` test** ([#10104](https://github.com/dotnet/aspire/pull/10104))
- **Parameter Improvements** ([#10094](https://github.com/dotnet/aspire/pull/10094))
- **Fix output when publish completes with warning** ([#10077](https://github.com/dotnet/aspire/pull/10077))
- **Add ContainerBuildOptions support to ResourceContainerImageBuilder for customizing dotnet publish** ([#10074](https://github.com/dotnet/aspire/pull/10074))
- **Add ProgressReporter property to DeployingContext** ([#10067](https://github.com/dotnet/aspire/pull/10067))
- **Fix AzurePublishingContext to ignore resources with ManifestPublishingCallbackAnnotation.Ignore** ([#10064](https://github.com/dotnet/aspire/pull/10064))
- **Enable PublishAot where available** ([#10053](https://github.com/dotnet/aspire/pull/10053))
- **Refactor publishing state model and CLI protocol to aggregate CompletionState at all levels** ([#10037](https://github.com/dotnet/aspire/pull/10037))
- **Localized file check-in by OneLocBuild Task: Build definition ID 1309: Build ID 2737112** ([#10028](https://github.com/dotnet/aspire/pull/10028))
- **Localized file check-in by OneLocBuild Task: Build definition ID 1309: Build ID 2736020** ([#10006](https://github.com/dotnet/aspire/pull/10006))
- **Add extension methods to PublishingStep & PublishingTask for direct Complete/Update operations** ([#9995](https://github.com/dotnet/aspire/pull/9995))
- **Improve the publish/deploy output** ([#9981](https://github.com/dotnet/aspire/pull/9981))
- **Add isError parameter to CompleteStepAsync method in PublishingActivityProgressReporter** ([#9979](https://github.com/dotnet/aspire/pull/9979))
- **Update PublishingActivityProgressReporter to support step/task views** ([#9971](https://github.com/dotnet/aspire/pull/9971))
- **Inject DOTNET_CLI_USE_MSBUILD_SERVER env var for apphost builds and runs, using configuration for value** ([#9946](https://github.com/dotnet/aspire/pull/9946))
- **Emit Int32OrStringV1 as int from Kubernetes publisher.** ([#9933](https://github.com/dotnet/aspire/pull/9933))
- **Fix Kubernetes publisher Kubernetes and Helm resource names** ([#9898](https://github.com/dotnet/aspire/pull/9898))
- **Quarantine flaky test PublishCommandSucceedsEndToEnd** ([#9872](https://github.com/dotnet/aspire/pull/9872))
- **Fail on publish/deploy if nothing happened.** ([#9850](https://github.com/dotnet/aspire/pull/9850))
- **Add Kubernetes publisher Workload and Resource extension points** ([#9819](https://github.com/dotnet/aspire/pull/9819))
- **Add support for DeployCommand and DeployingCallbackAnnotation** ([#9792](https://github.com/dotnet/aspire/pull/9792))
- **Localized file check-in by OneLocBuild Task: Build definition ID 1309: Build ID 2727100** ([#9761](https://github.com/dotnet/aspire/pull/9761))
- **Fix Azure Container Apps deployment failure with uppercase resource names** ([#9752](https://github.com/dotnet/aspire/pull/9752))
- **Add generic WithEnvironment overload for IValueProvider and IManifestExpressionProvider** ([#9748](https://github.com/dotnet/aspire/pull/9748))
- **[CI] Publish binaries for Aspire.Cli** ([#9695](https://github.com/dotnet/aspire/pull/9695))
- **Add support for existing Log Analytics Workspace when publishing into a Container App Environment** ([#9624](https://github.com/dotnet/aspire/pull/9624))
- **Externalize unknown parameters in ContainerApps and AppServiceWebSite** ([#9619](https://github.com/dotnet/aspire/pull/9619))
- **Remove a few auto injected known parameters** ([#9590](https://github.com/dotnet/aspire/pull/9590))
- **[release/9.3] Use ProcessSpec for invoking dotnet publish** ([#9561](https://github.com/dotnet/aspire/pull/9561))
- **Update the WithInitFiles implementations to use bind mounts in publish mode** ([#9528](https://github.com/dotnet/aspire/pull/9528))
- **Localized file check-in by OneLocBuild Task: Build definition ID 1309: Build ID 2719391** ([#9469](https://github.com/dotnet/aspire/pull/9469))

## Provisioning and Deployment

Aspire 9.4 expands support for Azure provisioning, including enhancements to Azure AI, OpenAI, SQL, and custom scripts. These updates make it easier to configure secure cloud resources with appropriate defaults and role assignments, streamlining both local development and production rollouts.

- **[release/9.4] Refactor Azure Storage API** ([#10303](https://github.com/dotnet/aspire/pull/10303))
- **Change Azure OpenAI to use CognitiveServicesOpenAIUser role by default** ([#10293](https://github.com/dotnet/aspire/pull/10293))
- **Add role support to Azure AI Foundry** ([#10292](https://github.com/dotnet/aspire/pull/10292))
- **Update AzureSqlServerResource to use the new AzurePowerShellScript provisioning feature** ([#10290](https://github.com/dotnet/aspire/pull/10290))
- **Refactor Azure Storage API** ([#10241](https://github.com/dotnet/aspire/pull/10241))
- **Refactor Endpoint bicep references in AI Foundry resource** ([#10193](https://github.com/dotnet/aspire/pull/10193))
- **Kubernetes Publisher connection strings fix** ([#10091](https://github.com/dotnet/aspire/pull/10091))
- **Add Azure AI Foundry support** ([#9974](https://github.com/dotnet/aspire/pull/9974))
- **[release/9.3] Fix SqlServer PowerShell module version to avoid breaking changes in 22.4.5.1** ([#9958](https://github.com/dotnet/aspire/pull/9958))
- **Fix SqlServer PowerShell module version to avoid breaking changes in 22.4.5.1** ([#9939](https://github.com/dotnet/aspire/pull/9939))
- **Add connection string to resource details** ([#9882](https://github.com/dotnet/aspire/pull/9882))
- **Use custom ConnectionStringBuilder when mutating connection strings** ([#9839](https://github.com/dotnet/aspire/pull/9839))
- **[release/9.3] Force SqlDatabase resource api version** ([#9535](https://github.com/dotnet/aspire/pull/9535))
- **Force SqlDatabase resource api version** ([#9530](https://github.com/dotnet/aspire/pull/9530))
- **[release/9.3] Fix Blob Container Connection String Format Exception** ([#9496](https://github.com/dotnet/aspire/pull/9496))

## Other improvements

We've also made a variety of under-the-hood improvements to increase test reliability, enhance telemetry, and reduce visual clutter. These changes keep Aspire fast, clean, and ready for production workflows.

- **Branding updates for 9.5.0** ([#10302](https://github.com/dotnet/aspire/pull/10302))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#10291](https://github.com/dotnet/aspire/pull/10291))
- **Add missing end quote for Outputs declaration** ([#10289](https://github.com/dotnet/aspire/pull/10289))
- **bump the version of azure.provisioning** ([#10284](https://github.com/dotnet/aspire/pull/10284))
- **Use different names for persistent and session scoped networks** ([#10278](https://github.com/dotnet/aspire/pull/10278))
- **Add option to console logs to wrap log lines** ([#10271](https://github.com/dotnet/aspire/pull/10271))
- **Fix thread safety issue in Razor view** ([#10270](https://github.com/dotnet/aspire/pull/10270))
- **Fix typo in `TalkingClockResource` comment** ([#10269](https://github.com/dotnet/aspire/pull/10269))
- **Use log output channel and remove abstraction** ([#10256](https://github.com/dotnet/aspire/pull/10256))
- **Add VS Code instructions to contributing gudelines** ([#10250](https://github.com/dotnet/aspire/pull/10250))
- **YARP: do not defer configuration generation when using env variable** ([#10242](https://github.com/dotnet/aspire/pull/10242))
- **Subscribe to this specific resource, not all.** ([#10238](https://github.com/dotnet/aspire/pull/10238))
- **Replace french_fries emoji with check_mark in CLI template output** ([#10237](https://github.com/dotnet/aspire/pull/10237))
- **Decrease message bar button text size** ([#10234](https://github.com/dotnet/aspire/pull/10234))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#10230](https://github.com/dotnet/aspire/pull/10230))
- **Fix ready event subscription for Foundry resource** ([#10229](https://github.com/dotnet/aspire/pull/10229))
- **Suppress CS9881 in generated project/apphost metadata types** ([#10223](https://github.com/dotnet/aspire/pull/10223))
- **Fix starvs.cmd script** ([#10219](https://github.com/dotnet/aspire/pull/10219))
- **Installs aspire extension on codespaces automatically** ([#10217](https://github.com/dotnet/aspire/pull/10217))
- **[main] Update dependencies from dotnet/arcade** ([#10216](https://github.com/dotnet/aspire/pull/10216))
- **update extension readme, add contributing.md** ([#10215](https://github.com/dotnet/aspire/pull/10215))
- **[Automated] Update Playground Manifests** ([#10214](https://github.com/dotnet/aspire/pull/10214))
- **Configure YARP with environment variables** ([#10210](https://github.com/dotnet/aspire/pull/10210))
- **Add user-friendly message when AddCommand search returns no results** ([#10209](https://github.com/dotnet/aspire/pull/10209))
- **Refine aspire CLI run command grid display with right-aligned labels and padding** ([#10207](https://github.com/dotnet/aspire/pull/10207))
- **Update package.json, add extension logo and license** ([#10190](https://github.com/dotnet/aspire/pull/10190))
- **Clear status after error** ([#10187](https://github.com/dotnet/aspire/pull/10187))
- **Code clean up in VersionFetcher** ([#10180](https://github.com/dotnet/aspire/pull/10180))
- **Fix Foundry Local setup link** ([#10179](https://github.com/dotnet/aspire/pull/10179))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#10178](https://github.com/dotnet/aspire/pull/10178))
- **Ignore NU1510 warnings** ([#10176](https://github.com/dotnet/aspire/pull/10176))
- **Update Event Subscriptions in tests to use new `OnXYZ` methods** ([#10171](https://github.com/dotnet/aspire/pull/10171))
- **Add package ID filtering to VersionFetcher.GetLatestVersion for robustness** ([#10166](https://github.com/dotnet/aspire/pull/10166))
- **Fix logic for filtering packages coming back from GetPackagesAsync.** ([#10164](https://github.com/dotnet/aspire/pull/10164))
- **Fix dependabot alert by bumping version of braces resolved by yarn** ([#10153](https://github.com/dotnet/aspire/pull/10153))
- **Add NameOutputReference and AddAsExistingResource to AzureAppServiceEnvironmentResource** ([#10152](https://github.com/dotnet/aspire/pull/10152))
- **Drop "sample" from aspire new description** ([#10143](https://github.com/dotnet/aspire/pull/10143))
- **Fix console logs pause/resume flaky test** ([#10134](https://github.com/dotnet/aspire/pull/10134))
- **Fix Typos in TalkingClockResource** ([#10117](https://github.com/dotnet/aspire/pull/10117))
- **Update devcontainer to use .NET 10.0 preview image** ([#10115](https://github.com/dotnet/aspire/pull/10115))
- **Throw on no input in ExtensionBackchannel** ([#10114](https://github.com/dotnet/aspire/pull/10114))
- **Reapply "Update dependencies from microsoft/usvc-apiserver** ([#10112](https://github.com/dotnet/aspire/pull/10112))
- **Revert "Update dependencies from microsoft/usvc-apiserver** ([#10110](https://github.com/dotnet/aspire/pull/10110))
- **[main] Update dependencies from dotnet/arcade** ([#10100](https://github.com/dotnet/aspire/pull/10100))
- **Add `IResourceBuilder<T>` extension method for event subscriptions** ([#10097](https://github.com/dotnet/aspire/pull/10097))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#10096](https://github.com/dotnet/aspire/pull/10096))
- **Fix tests failing on Azdo** ([#10095](https://github.com/dotnet/aspire/pull/10095))
- **Migrate to slnx, but also add missing project references** ([#10092](https://github.com/dotnet/aspire/pull/10092))
- **Quarantine flaky test StartResourceForcesStart** ([#10089](https://github.com/dotnet/aspire/pull/10089))
- **Cleanup error handling conditions for compute environments** ([#10078](https://github.com/dotnet/aspire/pull/10078))
- **Update to .NET 10 SDK and Arcade 10** ([#10075](https://github.com/dotnet/aspire/pull/10075))
- **Yarp: defer configuration generation** ([#10072](https://github.com/dotnet/aspire/pull/10072))
- **Redirect cli logger output to output channel and improve extension logging. Recreate terminal on first command run & then reuse** ([#10071](https://github.com/dotnet/aspire/pull/10071))
- **[Automated] Update Playground Manifests** ([#10070](https://github.com/dotnet/aspire/pull/10070))
- **Fix `YarpCluster` construction and add constructor that takes `IResourceWithServiceDiscovery`** ([#10060](https://github.com/dotnet/aspire/pull/10060))
- **Fix DevcontainerSettingsWriter directory creation in clean codespace environments** ([#10059](https://github.com/dotnet/aspire/pull/10059))
- **Dedupe shared backchannel types** ([#10058](https://github.com/dotnet/aspire/pull/10058))
- **[main] Update dependencies from dotnet/arcade** ([#10055](https://github.com/dotnet/aspire/pull/10055))
- **More extension cleanup** ([#10054](https://github.com/dotnet/aspire/pull/10054))
- **Only ask for distinct package names in aspire add** ([#10052](https://github.com/dotnet/aspire/pull/10052))
- **[CI] Reduce Outerloop tests workflow to run once every 6 hours, instead of every 2** ([#10044](https://github.com/dotnet/aspire/pull/10044))
- **bump provisioning libraries to latest versions** ([#10042](https://github.com/dotnet/aspire/pull/10042))
- **Add new YARP fluent configuration** ([#10040](https://github.com/dotnet/aspire/pull/10040))
- **Add dot notation support for aspire config commands** ([#10035](https://github.com/dotnet/aspire/pull/10035))
- **Add Aspire upgrade check** ([#10034](https://github.com/dotnet/aspire/pull/10034))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#10031](https://github.com/dotnet/aspire/pull/10031))
- **Update manifest-spec.md** ([#10025](https://github.com/dotnet/aspire/pull/10025))
- **[main] Update dependencies from dotnet/arcade** ([#10021](https://github.com/dotnet/aspire/pull/10021))
- **Improve console logs error handling** ([#10012](https://github.com/dotnet/aspire/pull/10012))
- **Fix TFM/Aspire version selection in templates** ([#10009](https://github.com/dotnet/aspire/pull/10009))
- **Increase BrowserTokenAuthenticationTests timeouts** ([#10007](https://github.com/dotnet/aspire/pull/10007))
- **Exclude FluentUI from update dependencies** ([#9998](https://github.com/dotnet/aspire/pull/9998))
- **Use a digit in resx string which is passed to string.Format** ([#9996](https://github.com/dotnet/aspire/pull/9996))
- **Use STJ source generation in Cli backchannel** ([#9993](https://github.com/dotnet/aspire/pull/9993))
- **Add CLI version update notifications to Aspire CLI** ([#9992](https://github.com/dotnet/aspire/pull/9992))
- **Stream apphost logs across backchannel.** ([#9990](https://github.com/dotnet/aspire/pull/9990))
- **Clean up extension** ([#9988](https://github.com/dotnet/aspire/pull/9988))
- **Revert to FluentUI 4.11.9 again** ([#9987](https://github.com/dotnet/aspire/pull/9987))
- **Fix support for non-localhost endpoint targets** ([#9977](https://github.com/dotnet/aspire/pull/9977))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9975](https://github.com/dotnet/aspire/pull/9975))
- **Add .http file to ApiService in starter template** ([#9973](https://github.com/dotnet/aspire/pull/9973))
- **Bump System.CommandLine.** ([#9968](https://github.com/dotnet/aspire/pull/9968))
- **Add ExternalServiceResource for modeling external services with service discovery support** ([#9965](https://github.com/dotnet/aspire/pull/9965))
- **Bumping patch version for 9.3.2** ([#9963](https://github.com/dotnet/aspire/pull/9963))
- **Update .NET dependencies** ([#9962](https://github.com/dotnet/aspire/pull/9962))
- **Reorder & don't remember Aspire version in IDE's** ([#9961](https://github.com/dotnet/aspire/pull/9961))
- **Update launchSettings.json to support manifest generation** ([#9959](https://github.com/dotnet/aspire/pull/9959))
- **Refactor settings file path handling** ([#9951](https://github.com/dotnet/aspire/pull/9951))
- **bump the version of azure.provisioning.operationalinsights** ([#9947](https://github.com/dotnet/aspire/pull/9947))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9944](https://github.com/dotnet/aspire/pull/9944))
- **[build] Fix versions of arcade dependencies** ([#9941](https://github.com/dotnet/aspire/pull/9941))
- **Fix glob pattern so it includes tests/Directory.Packages.props in update-dependencies workflow** ([#9940](https://github.com/dotnet/aspire/pull/9940))
- **[Automated] Update Playground Manifests** ([#9936](https://github.com/dotnet/aspire/pull/9936))
- **Add k8s tooling.** ([#9925](https://github.com/dotnet/aspire/pull/9925))
- **Add a section on requirements for building the repo on Alpine Linux** ([#9924](https://github.com/dotnet/aspire/pull/9924))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9923](https://github.com/dotnet/aspire/pull/9923))
- **Main** ([#9920](https://github.com/dotnet/aspire/pull/9920))
- **Quarantine flaky test TracingEnablesTheRightActivitySource_Keyed** ([#9918](https://github.com/dotnet/aspire/pull/9918))
- **bump versions of azure.provisioning libraries** ([#9915](https://github.com/dotnet/aspire/pull/9915))
- **Fix console logs clear not going back to a blank state** ([#9910](https://github.com/dotnet/aspire/pull/9910))
- **clean up rpc server connection logic, update string** ([#9907](https://github.com/dotnet/aspire/pull/9907))
- **Fix Outerloop Tests workflow failing on forks** ([#9903](https://github.com/dotnet/aspire/pull/9903))
- **Add linux-musl-amd64 (Alpine linux) specific build support** ([#9901](https://github.com/dotnet/aspire/pull/9901))
- **Update MTP to 1.7.2** ([#9895](https://github.com/dotnet/aspire/pull/9895))
- **Update xunit.v3 to 3.0.0-pre.25** ([#9894](https://github.com/dotnet/aspire/pull/9894))
- **Disable local auth on Azure SignalR** ([#9891](https://github.com/dotnet/aspire/pull/9891))
- **Template updates for Aspire 9.4 and .NET 10** ([#9883](https://github.com/dotnet/aspire/pull/9883))
- **Update xunit.v3 to 2.0.3** ([#9876](https://github.com/dotnet/aspire/pull/9876))
- **Improve compatibility error message to include Aspire.Hosting package version** ([#9875](https://github.com/dotnet/aspire/pull/9875))
- **Revert "update fluentui packages (#9794)"** ([#9873](https://github.com/dotnet/aspire/pull/9873))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9864](https://github.com/dotnet/aspire/pull/9864))
- **Consolidate Aspire CLI config subcommands into single command with verb argument** ([#9849](https://github.com/dotnet/aspire/pull/9849))
- **Use single ActivitySource across CLI components** ([#9848](https://github.com/dotnet/aspire/pull/9848))
- **Start the beginning of AOT'ing the Aspire.Cli.** ([#9841](https://github.com/dotnet/aspire/pull/9841))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9840](https://github.com/dotnet/aspire/pull/9840))
- **Cleanup error handling conditions for ACA compute environment** ([#9838](https://github.com/dotnet/aspire/pull/9838))
- **Quarantine flaky ResourceCommandServiceTests** ([#9836](https://github.com/dotnet/aspire/pull/9836))
- **[Automated] Update Playground Manifests** ([#9830](https://github.com/dotnet/aspire/pull/9830))
- **Enhance AppHostExitsWhenCliProcessPidDies test diagnostics and add quarantined test filtering documentation** ([#9816](https://github.com/dotnet/aspire/pull/9816))
- **Fix user secrets duplication issue by normalizing to flat configuration format** ([#9815](https://github.com/dotnet/aspire/pull/9815))
- **Fix CLI to automatically fallback when apphost file in settings doesn't exist** ([#9814](https://github.com/dotnet/aspire/pull/9814))
- **Quarantine flaky test WithHttpCommand_EnablesCommandUsingCustomUpdateStateCallback** ([#9813](https://github.com/dotnet/aspire/pull/9813))
- **Menu button concurrency fix** ([#9806](https://github.com/dotnet/aspire/pull/9806))
- **Add ResourceCommandService** ([#9805](https://github.com/dotnet/aspire/pull/9805))
- **Quarantine flaky test WithHttpCommand_UsesNamedHttpClient** ([#9802](https://github.com/dotnet/aspire/pull/9802))
- **Disable local auth on Azure EventHubs and WebPubSub** ([#9799](https://github.com/dotnet/aspire/pull/9799))
- **Update fluentui packages** ([#9794](https://github.com/dotnet/aspire/pull/9794))
- **Rename unset so that it gets re-translated** ([#9793](https://github.com/dotnet/aspire/pull/9793))
- **Quarantine flaky test WithHttpCommand_CallsPrepareRequestCallback_BeforeSendingRequest** ([#9791](https://github.com/dotnet/aspire/pull/9791))
- **[Automated] Update Playground Manifests** ([#9788](https://github.com/dotnet/aspire/pull/9788))
- **Fix ACA DataProtection** ([#9786](https://github.com/dotnet/aspire/pull/9786))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9780](https://github.com/dotnet/aspire/pull/9780))
- **Set functionapp kind on Azure Functions projects** ([#9779](https://github.com/dotnet/aspire/pull/9779))
- **Make verify tool accessible to the coding agent** ([#9778](https://github.com/dotnet/aspire/pull/9778))
- **Default ACA autoConfigureDataProtection to true for .NET projects** ([#9777](https://github.com/dotnet/aspire/pull/9777))
- **Quarantine flaky test WithHttpCommand_CallsGetResponseCallback_AfterSendingRequest** ([#9776](https://github.com/dotnet/aspire/pull/9776))
- **Revert "Make verify tool accessible to the coding agent (#9753)"** ([#9775](https://github.com/dotnet/aspire/pull/9775))
- **Allow YARP to be configured in a programmatic way** ([#9766](https://github.com/dotnet/aspire/pull/9766))
- **[main] Update dependencies from dotnet/arcade** ([#9760](https://github.com/dotnet/aspire/pull/9760))
- **Improve/Fix hierarchical partition keys for Azure Cosmos** ([#9758](https://github.com/dotnet/aspire/pull/9758))
- **Default to single resource on console logs and metrics pages** ([#9757](https://github.com/dotnet/aspire/pull/9757))
- **Make verify tool accessible to the coding agent** ([#9753](https://github.com/dotnet/aspire/pull/9753))
- **Fix error CSS** ([#9750](https://github.com/dotnet/aspire/pull/9750))
- **Start localizing CLI** ([#9741](https://github.com/dotnet/aspire/pull/9741))
- **Simplify AzureProvisioner and make it testable by removing unnecessary abstraction layers** ([#9737](https://github.com/dotnet/aspire/pull/9737))
- **Disable failing test WithHttpCommand_ResultsInExpectedResultForHttpMethod** ([#9730](https://github.com/dotnet/aspire/pull/9730))
- **more instructions** ([#9726](https://github.com/dotnet/aspire/pull/9726))
- **Quarantine flaky test CanOverrideLaunchProfileViaArgsAdHocBuilder** ([#9721](https://github.com/dotnet/aspire/pull/9721))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9720](https://github.com/dotnet/aspire/pull/9720))
- **[release/9.3] Skip role assignment handling for emulators (#9705)** ([#9716](https://github.com/dotnet/aspire/pull/9716))
- **Use a persistent container network if there are persistent container resources** ([#9715](https://github.com/dotnet/aspire/pull/9715))
- **Speed up/robustify copilot setup step** ([#9710](https://github.com/dotnet/aspire/pull/9710))
- **Skip role assignment handling for emulators** ([#9705](https://github.com/dotnet/aspire/pull/9705))
- **Add IAzureComputeEnvironmentResource interface for Azure-backed compute** ([#9700](https://github.com/dotnet/aspire/pull/9700))
- **Ensure resource non-endpoint URLs are active when initialized** ([#9696](https://github.com/dotnet/aspire/pull/9696))
- **CosmosDB: Add the 'EnableServerless' capability to accounts** ([#9692](https://github.com/dotnet/aspire/pull/9692))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9689](https://github.com/dotnet/aspire/pull/9689))
- **[release/9.3] Fix the state propagation for azure resources** ([#9687](https://github.com/dotnet/aspire/pull/9687))
- **Fix the state propagation for azure resources** ([#9686](https://github.com/dotnet/aspire/pull/9686))
- **Launch cli with proper params, remove unused strings and old command logic** ([#9684](https://github.com/dotnet/aspire/pull/9684))
- **bump version of azure.provisioning to 1.0.1** ([#9680](https://github.com/dotnet/aspire/pull/9680))
- **Add missing imports, move rpc server test file** ([#9678](https://github.com/dotnet/aspire/pull/9678))
- **Add aspire config commands for managing configuration settings** ([#9676](https://github.com/dotnet/aspire/pull/9676))
- **Quarantine some flaky tests** ([#9675](https://github.com/dotnet/aspire/pull/9675))
- **Update Docker Image tags** ([#9674](https://github.com/dotnet/aspire/pull/9674))
- **ParallelOptions not being passed to Parallel.ForEachAsync in FindAppHostProjectFilesAsync** ([#9661](https://github.com/dotnet/aspire/pull/9661))
- **Update using-latest-daily.md** ([#9658](https://github.com/dotnet/aspire/pull/9658))
- **Rename WithBrowserPort to WithHostPort for consistency with other hosting packages** ([#9657](https://github.com/dotnet/aspire/pull/9657))
- **Fix thread safety issue in FindAppHostProjectFilesAsync by switching to ConcurrentBag** ([#9655](https://github.com/dotnet/aspire/pull/9655))
- **[CI] Use ⚠️emoji for zero tests run** ([#9654](https://github.com/dotnet/aspire/pull/9654))
- **Add definitions for new log streaming options** ([#9644](https://github.com/dotnet/aspire/pull/9644))
- **change 2->8 cores for copilot setup build** ([#9626](https://github.com/dotnet/aspire/pull/9626))
- **Update verify and remove AddTextExtension** ([#9625](https://github.com/dotnet/aspire/pull/9625))
- **Add support for containers with Dockerfile to AzureAppServiceEnvironmentResource** ([#9620](https://github.com/dotnet/aspire/pull/9620))
- **Add GetSecret convenience API and WithSecret methods for AzureKeyVaultResource** ([#9615](https://github.com/dotnet/aspire/pull/9615))
- **[Automated] Update Playground Manifests** ([#9610](https://github.com/dotnet/aspire/pull/9610))
- **Only expose endpoint port in docker compose if external is set to true** ([#9604](https://github.com/dotnet/aspire/pull/9604))
- **Revert "Add definitions for new log streaming options (#9593)"** ([#9601](https://github.com/dotnet/aspire/pull/9601))
- **Remove dependencies on AfterEndpointsAllocatedEvent** ([#9598](https://github.com/dotnet/aspire/pull/9598))
- **Add definitions for new log streaming options** ([#9593](https://github.com/dotnet/aspire/pull/9593))
- **[CI] Collect dcp logs on Outerloop workflow** ([#9584](https://github.com/dotnet/aspire/pull/9584))
- **Initial Aspire extension code** ([#9578](https://github.com/dotnet/aspire/pull/9578))
- **ci: set commit-message in refresh-manifests workflow** ([#9574](https://github.com/dotnet/aspire/pull/9574))
- **Remove 37 unused resources from .resx files** ([#9573](https://github.com/dotnet/aspire/pull/9573))
- **Allow scheme to be specified when configuring Azurite** ([#9570](https://github.com/dotnet/aspire/pull/9570))
- **[Automated] Update Manifests** ([#9562](https://github.com/dotnet/aspire/pull/9562))
- **Improve app host search messaging and add spacing for better UX** ([#9552](https://github.com/dotnet/aspire/pull/9552))
- **[main] Update dependencies from microsoft/usvc-apiserver** ([#9549](https://github.com/dotnet/aspire/pull/9549))
- **[Automated] Update Manifests** ([#9548](https://github.com/dotnet/aspire/pull/9548))
- **Rename Aspire.Components.Common.Tests to Aspire.Components.Common.TestUtilities** ([#9547](https://github.com/dotnet/aspire/pull/9547))
- **AppHost disambiguation and selection.** ([#9542](https://github.com/dotnet/aspire/pull/9542))
- **[release/9.3] branding for 9.3.1** ([#9539](https://github.com/dotnet/aspire/pull/9539))
- **Fix Dev Container / Codespaces support for non-default usernames** ([#9538](https://github.com/dotnet/aspire/pull/9538))
- **[release/9.3] fix markdown lint in release/9.3** ([#9536](https://github.com/dotnet/aspire/pull/9536))
- **Split Azure tests by resource in Aspire.Hosting.Azure.Tests** ([#9527](https://github.com/dotnet/aspire/pull/9527))
- **[Automated] Update Manifests** ([#9525](https://github.com/dotnet/aspire/pull/9525))
- **TestShop: Add IsRunMode check for database key** ([#9522](https://github.com/dotnet/aspire/pull/9522))
- **Update refresh-manifests.yml** ([#9514](https://github.com/dotnet/aspire/pull/9514))
- **Support hierarchical partition keys for Azure Cosmos** ([#9513](https://github.com/dotnet/aspire/pull/9513))
- **Allow mounting the docker socket using WithBindMount** ([#9511](https://github.com/dotnet/aspire/pull/9511))
- **The --source argument is not preserved when running aspire add -s** ([#9509](https://github.com/dotnet/aspire/pull/9509))
- **[main] Update dependencies from dotnet/arcade** ([#9507](https://github.com/dotnet/aspire/pull/9507))
- **Repo MCP tools** ([#9506](https://github.com/dotnet/aspire/pull/9506))
- **Change .dotnet/aspire to .aspire in temporary working files** ([#9505](https://github.com/dotnet/aspire/pull/9505))
- **Automate refreshing manifests with GitHub Action** ([#9503](https://github.com/dotnet/aspire/pull/9503))
- **Expose the NameOutputReference property on AzureResources** ([#9501](https://github.com/dotnet/aspire/pull/9501))
- **Drop support for hybrid mode Azure Container Apps** ([#9500](https://github.com/dotnet/aspire/pull/9500))
- **Fix malformed table output in aspire run command when no resources are present** ([#9498](https://github.com/dotnet/aspire/pull/9498))
- **Remove leftover ElasticSearch references after moving ElasticSearch out of repo** ([#9494](https://github.com/dotnet/aspire/pull/9494))
- **[WIP] Aspire CLI ctrl+c error message** ([#9491](https://github.com/dotnet/aspire/pull/9491))
- **Use orchestrator container delay start feature for better `WithExplictStart` + `ContainerLifetime.Persistent` support** ([#9471](https://github.com/dotnet/aspire/pull/9471))
- **[CI] Use the local test report generator for Outerloop workflow** ([#9439](https://github.com/dotnet/aspire/pull/9439))
- **Bump to latest azure-functions-core-tools** ([#9267](https://github.com/dotnet/aspire/pull/9267))
- **[Automated] Update dependencies** ([#9252](https://github.com/dotnet/aspire/pull/9252))

-->
