---
title: Connect an ASP.NET Core app to SQL Server using .NET Aspire and Entity Framework Core
description: Learn how to connect an ASP.NET Core app to to SQL Server using .NET Aspire and Entity Framework Core.
ms.date: 12/02/2024
ms.topic: tutorial
---

# Tutorial: Connect an ASP.NET Core app to SQL Server using .NET Aspire and Entity Framework Core

In this tutorial, you create an ASP.NET Core app that uses a .NET Aspire Entity Framework Core SQL Server integration to connect to SQL Server to read and write support ticket data. [Entity Framework Core](/ef/core/) is a lightweight, extensible, open source object-relational mapper that enables .NET developers to work with databases using .NET objects. You'll learn how to:

> [!div class="checklist"]
>
> - Create a basic .NET app that is set up to use .NET Aspire integrations
> - Add a .NET Aspire integration to connect to SQL Server
> - Configure and use .NET Aspire Component features to read and write from the database

[!INCLUDE [aspire-prereqs](../includes/aspire-prereqs.md)]

## Create the sample solution

1. At the top of Visual Studio, navigate to **File** > **New** > **Project**.
1. In the dialog window, search for *Blazor* and select **Blazor Web App**. Choose **Next**.
1. On the **Configure your new project** screen:
    - Enter a **Project Name** of **AspireSQLEFCore**.
    - Leave the rest of the values at their defaults and select **Next**.
1. On the **Additional information** screen:
    - Make sure **.NET 9.0** is selected.
    - Ensure the **Interactive render mode** is set to **None**.
    - Check the **Enlist in .NET Aspire orchestration** option and select **Create**.

Visual Studio creates a new ASP.NET Core solution that is structured to use .NET Aspire. The solution consists of the following projects:

- **AspireSQLEFCore**: A Blazor project that depends on service defaults.
- **AspireSQLEFCore.AppHost**: An orchestrator project designed to connect and configure the different projects and services of your app. The orchestrator should be set as the startup project.
- **AspireSQLEFCore.ServiceDefaults**: A shared class library to hold configurations that can be reused across the projects in your solution.

## Create the database model and context classes

To represent a user submitted support request, add the following `SupportTicket` model class at the root of the _AspireSQLEFCore_ project.

:::code source="snippets/tutorial/AspireSQLEFCore/AspireSQLEFCore/SupportTicket.cs":::

Add the following `TicketDbContext` data context class at the root of the **AspireSQLEFCore** project. The class inherits <xref:System.Data.Entity.DbContext?displayProperty=fullName> to work with Entity Framework and represent your database.

:::code source="snippets/tutorial/AspireSQLEFCore/AspireSQLEFCore/TicketContext.cs":::

## Add the .NET Aspire integration to the Blazor app

Add the [.NET Aspire Entity Framework Core Sql Server library package](sql-server-entity-framework-integration.md) to your _AspireSQLEFCore_ project:

```dotnetcli
dotnet add package Aspire.Microsoft.EntityFrameworkCore.SqlServer
```

Your _AspireSQLEFCore_ project is now set up to use .NET Aspire integrations. Here's the updated _AspireSQLEFCore.csproj_ file:

:::code language="xml" source="snippets/tutorial/AspireSQLEFCore/AspireSQLEFCore/AspireSQLEFCore.csproj" highlight="10":::

## Configure the .NET Aspire integration

In the _:::no-loc text="Program.cs":::_ file of the _AspireSQLEFCore_ project, add a call to the <xref:Microsoft.Extensions.Hosting.AspireSqlServerEFCoreSqlClientExtensions.AddSqlServerDbContext%2A> extension method after the creation of the `builder` but before the call to `AddServiceDefaults`. For more information, see [.NET Aspire service defaults](../fundamentals/service-defaults.md). Provide the name of your connection string as a parameter.

:::code language="csharp" source="snippets/tutorial/AspireSQLEFCore/AspireSQLEFCore/Program.cs" range="1-14" highlight="5":::

This method accomplishes the following tasks:

- Registers a `TicketContext` with the DI container for connecting to the containerized Azure SQL Database.
- Automatically enable corresponding health checks, logging, and telemetry.

## Create the database

While developing locally, you need to create a database inside the SQL Server container. Update the _:::no-loc text="Program.cs":::_ file with the following code:

:::code language="csharp" source="snippets/tutorial/AspireSQLEFCore/AspireSQLEFCore/Program.cs" range="1-30" highlight="16-30":::

The preceding code:

- Checks if the app is running in a development environment.
- If it is, it retrieves the `TicketContext` service from the DI container and calls `Database.EnsureCreated()` to create the database if it doesn't already exist.

> [!NOTE]
> Note that `EnsureCreated()` is not suitable for production environments, and it only creates the database as defined in the context. It doesn't apply any migrations. For more information on Entity Framework Core migrations in .NET Aspire, see [Apply Entity Framework Core migrations in .NET Aspire](ef-core-migrations.md).

## Create the form

The app requires a form for the user to be able to submit support ticket information and save the entry to the database.

Use the following Razor markup to create a basic form, replacing the contents of the _Home.razor_ file in the _AspireSQLEFCore/Components/Pages_ directory:

:::code language="razor" source="snippets/tutorial/AspireSQLEFCore/AspireSQLEFCore/Components/Pages/Home.razor":::

For more information about creating forms in Blazor, see [ASP.NET Core Blazor forms overview](/aspnet/core/blazor/forms).

## Configure the AppHost

The _AspireSQLEFCore.AppHost_ project is the orchestrator for your app. It's responsible for connecting and configuring the different projects and services of your app. The orchestrator should be set as the startup project.

Add the [.NET Aspire Hosting Sql Server](sql-server-entity-framework-integration.md#hosting-integration) NuGet package to your _AspireStorage.AppHost_ project:

```dotnetcli
dotnet add package Aspire.Hosting.SqlServer
```

Replace the contents of the _:::no-loc text="Program.cs":::_ file in the _AspireSQLEFCore.AppHost_ project with the following code:

:::code language="csharp" source="snippets/tutorial/AspireSQLEFCore/AspireSQLEFCore.AppHost/Program.cs":::

The preceding code adds a SQL Server Container resource to your app and configures a connection to a database called `sqldata`. The Entity Framework classes you configured earlier will automatically use this connection when migrating and connecting to the database.

## Run and test the app locally

The sample app is now ready for testing. Verify that the submitted form data is persisted to the database by completing the following steps:

1. Select the run button at the top of Visual Studio (or <kbd>F5</kbd>) to launch your .NET Aspire project dashboard in the browser.
1. On the projects page, in the **AspireSQLEFCore** row, click the link in the **Endpoints** column to open the UI of your app.

    :::image type="content" source="media/app-home-screen.png" lightbox="media/app-home-screen.png" alt-text="A screenshot showing the home page of the .NET Aspire support application.":::

1. Enter sample data into the `Title` and `Description` form fields.
1. Select the **Submit** button, and the form submits the support ticket for processing — (then select **Clear** to clear the form).
1. The data you submitted displays in the table at the bottom of the page when the page reloads.
1. Close the web browser tabs that display the **AspireSQL** web app and the .NET Aspire dashboard.
1. Switch to Visual Studio and, to stop debugging, select the stop button or press <kbd>Shift + F5</kbd>.
1. To start debugging a second time, select the run button at the top of Visual Studio (or <kbd>F5</kbd>).
1. In the .NET Aspire dashboard, on the projects page, in the **AspireSQLEFCore** row, click the link in the **Endpoints** column to open the UI of your app.
1. Notice that the page doesn't display the ticket you created in the previous run.
1. Close the web browser tabs that display the **AspireSQL** web app and the .NET Aspire dashboard.
1. Switch to Visual Studio and, to stop debugging, select the stop button or press <kbd>Shift + F5</kbd>.

## Persist data across restarts

Developers often prefer their data to persist across restarts in the development environment for a more realistic database to run code against. To implement persistence in .NET Aspire, use the <xref:Aspire.Hosting.SqlServerBuilderExtensions.WithDataVolume*> method. This methods adds a Docker volume to your database container, which won't be destroyed every time you restart debugging.

1. In Visual Studio, in the _AspireSQLEFCore.AppHost_ project, double-click the _Program.cs_ code file.
1. Locate the following code:

    ```csharp
    var sql = builder.AddSqlServer("sql")
                     .AddDatabase("sqldata");
    ```

1. Modify that code to match the following:

    ```csharp
    var sql = builder.AddSqlServer("sql")
                     .WithDataVolume()
                     .AddDatabase("sqldata");
    ```

## Run and test the data persistence

Let's examine how the data volume changes the behavior of the solution:

1. Select the run button at the top of Visual Studio (or <kbd>F5</kbd>) to launch your .NET Aspire project dashboard in the browser.
1. On the projects page, in the **AspireSQLEFCore** row, click the link in the **Endpoints** column to open the UI of your app.
1. Enter sample data into the `Title` and `Description` form fields.
1. Select the **Submit** button, and the form submits the support ticket for processing — (then select **Clear** to clear the form).
1. The data you submitted displays in the table at the bottom of the page when the page reloads.
1. Close the web browser tabs that display the **AspireSQL** web app and the .NET Aspire dashboard.
1. Switch to Visual Studio and, to stop debugging, select the stop button or press <kbd>Shift + F5</kbd>.
1. To start debugging a second time, select the run button at the top of Visual Studio (or <kbd>F5</kbd>).
1. In the .NET Aspire dashboard, on the projects page, in the **AspireSQLEFCore** row, click the link in the **Endpoints** column to open the UI of your app.
1. Notice that the page now displays the ticket you created in the previous run.

## See also

- [.NET Aspire with SQL Database deployment](sql-server-integration-deployment.md)
- [.NET Aspire deployment via Azure Container Apps](../deployment/azure/aca-deployment.md)
- [Deploy a .NET Aspire project using GitHub Actions](../deployment/azure/aca-deployment-github-actions.md)
