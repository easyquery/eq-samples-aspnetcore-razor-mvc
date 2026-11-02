# EasyQuery samples for ASP.NET Core Razor Pages and MVC

[![NuGet](https://img.shields.io/nuget/v/Korzh.EasyQuery.AspNetCore)](https://www.nuget.org/packages/Korzh.EasyQuery.AspNetCore) [![npm](https://img.shields.io/npm/v/@easyquery/ui)](https://www.npmjs.com/package/@easyquery/ui) [![Build](https://github.com/easyquery/eq-samples-aspnetcore-razor-mvc/actions/workflows/build.yml/badge.svg)](https://github.com/easyquery/eq-samples-aspnetcore-razor-mvc/actions/workflows/build.yml) [![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

[EasyQuery](https://korzh.com/easyquery) adds a query builder, ad-hoc reporting and data filtering
to .NET applications. The samples in this repository show it on ASP.NET Core Razor Pages and MVC.

## Samples

| Sample | Scenario | Stack |
|---|---|---|
| [Advanced search](Razor.AdvancedSearch) | Advanced search | ASP.NET Core 10 · Razor Pages · C# |
| [Ad-hoc reporting](Razor.AdHocReporting) | Ad-hoc reporting | ASP.NET Core 8 · Razor Pages · C# |
| [Ad-hoc reporting (TypeScript)](RazorTypeScript.AdHocReporting) | Ad-hoc reporting | ASP.NET Core 8 · Razor Pages + TypeScript · C#, TypeScript |
| [Data filtering](Razor.DataFiltering) | Data filtering | ASP.NET Core 8 · Razor Pages · C# |
| [Data filtering](Mvc.DataFiltering) | Data filtering | ASP.NET Core 8 · MVC · C# |
| [Data filtering with authentication](Mvc.DataFiltering.Auth) | Data filtering | ASP.NET Core 8 · MVC · C# |

* **Advanced search.** A full-page query builder: users pick the columns, combine any conditions they need, run the query and work with the result — sorting, paging and export.
* **Ad-hoc reporting.** A basic ad-hoc reporting tool: users keep a list of reports, set up the columns and conditions of each one, and see the result as a table (with grouping and totals) or a chart.
* **Data filtering.** Filtering and paging added to an existing data page for one entity, without the EasyQuery middleware: a few elements on the page and a few lines in the controller or page model. Includes a simple full-text search.

## Quick start

1. Get the code: [download the ZIP](https://github.com/easyquery/eq-samples-aspnetcore-razor-mvc/archive/refs/heads/main.zip),
   or `git clone https://github.com/easyquery/eq-samples-aspnetcore-razor-mvc.git`.
2. Open `Samples.AspNetCore.sln` in Visual Studio 2022 (17.2 or later).
3. Set the sample you want as the startup project and run it. Or, from the command line, `cd` into a sample folder and run `dotnet run`; each sample's README has the exact steps.

Every sample creates and fills its own demo database on the first start. That can take a minute
or two; later starts take a few seconds.

## Package versions

All NuGet package versions of this repository, EasyQuery and EasyData included, are set in one
file: [`Directory.Packages.props`](Directory.Packages.props) (NuGet central package
management, which needs Visual Studio 2022 17.2+ or the .NET 6.0.300+ SDK). To try another
EasyQuery version, change it there.

## License key

You don't need one to start. On the first run the sample shows a **No license** dialog: enter your
email, then the code we send to it, and you get a free 30-day trial key on the spot. The key is saved to `EasyQuery.lck`; restart the sample to apply it.

Already have a key? Put it into `appsettings.json` of the app that serves the EasyQuery API and the dialog won't appear:

```json
"EasyQuery": {
  "LicenseKey": "your key goes here"
}
```

Ready to buy? See the [licensing options on korzh.com](https://korzh.com/easyquery?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc#licensing).

## Next steps

The sample runs on its own demo database. From here there are two ways to get to your own data.

### Connect a sample to your database

The quickest way to see EasyQuery on your data: keep the sample as it is and swap the database
under it. Every sample reaches its data through EF Core — an `AppDbContext` with the Northwind
entities in `Models/NWind/` — and builds the EasyQuery data model from it, so the change is mostly
about that `DbContext` and its connection:

1. **Point the connection string at your database** in `appsettings.json`.
2. **Switch the EF Core provider** (`options.UseSqlite(...)` in `Program.cs`) to yours. The
   SQL Server lines are already there, commented out; for another database add its EF Core
   provider package.
3. **Replace the demo `DbContext`** with yours: your entity classes instead of `Models/NWind/`,
   your `DbSet`s in `Data/AppDbContext.cs`. No classes yet? `dotnet ef dbcontext scaffold`
   generates them from the database.
4. **Turn off the demo seeding** — the `EnsureDbInitialized...` call at the end of `Program.cs`.
   It creates the Northwind tables and fills them on the first start.
5. **Remove what refers to Northwind**: entity names in the EasyQuery settings, the predefined
   queries or reports in `App_Data`, the columns of a hand-written result list, and SQLite-specific
   SQL settings.

Samples that build the model from the whole `DbContext` (advanced search, ad-hoc reporting) show
all its entities at once. Data filtering samples work with one entity (`Orders`) and need its
replacement as well. Where the `DbContext` also holds the users and their saved queries, the
simplest way is to leave that one as it is and add a second `DbContext` for your data. Each sample
lists its own steps:

| Sample | Data model | Steps |
|---|---|---|
| Advanced search | Whole `DbContext`, SQL | [Razor.AdvancedSearch](Razor.AdvancedSearch/README.md#connect-the-sample-to-your-database) |
| Ad-hoc reporting | Whole `DbContext`, SQL; users and reports in the same DB | [Razor.AdHocReporting](Razor.AdHocReporting/README.md#connect-the-sample-to-your-database), [RazorTypeScript.AdHocReporting](RazorTypeScript.AdHocReporting/README.md#connect-the-sample-to-your-database) |
| Data filtering | One entity, LINQ | [Razor.DataFiltering](Razor.DataFiltering/README.md#connect-the-sample-to-your-database), [Mvc.DataFiltering](Mvc.DataFiltering/README.md#connect-the-sample-to-your-database) |
| Data filtering with authentication | One entity, LINQ; users and queries in the same DB | [Mvc.DataFiltering.Auth](Mvc.DataFiltering.Auth/README.md#connect-the-sample-to-your-database) |

### Add EasyQuery to your own project

To get the same page into an application you already have, repeat what the sample does, step by
step:

1. **Install the packages**: `Korzh.EasyQuery.AspNetCore`, plus the one for the query engine
   (`.EntityFrameworkCore.Relational` for SQL, `.Linq` for LINQ) and, if you need it, export.
2. **Add the server side**: `AddEasyQuery()` in the services and `MapEasyQuery(...)` in the
   request pipeline of `Program.cs` — or, in the MVC data filtering samples, a controller.
3. **Add the UI**: copy the page or view that holds the EasyQuery widgets, with its styles.
4. **Connect your data**: tell EasyQuery your `DbContext` (or the entity to filter). The queries
   then run through its connection, in the SQL of its provider.
5. **Choose where queries are stored**: files in `App_Data`, the session, or a table per user.
6. **Set the license key**, as described in [License key](#license-key). Until there is one,
   the page shows the **No license** dialog instead of the EasyQuery widgets.

The exact packages, code and files differ from sample to sample:

| Sample | What you copy | Steps |
|---|---|---|
| Advanced search | `EasyQueryConfigurator`, the page and its layout (for a first look, the simpler ready-made page of `Korzh.EasyQuery.RazorUI` needs no copying) | [Razor.AdvancedSearch](Razor.AdvancedSearch/README.md#add-easyquery-to-your-own-project) |
| Ad-hoc reporting | The `MapEasyQuery` settings, the page, the report store | [Razor.AdHocReporting](Razor.AdHocReporting/README.md#add-easyquery-to-your-own-project), [RazorTypeScript.AdHocReporting](RazorTypeScript.AdHocReporting/README.md#add-easyquery-to-your-own-project) (npm + TypeScript) |
| Data filtering (Razor Pages) | One `MapEasyQuery` call and a few elements on your page | [Razor.DataFiltering](Razor.DataFiltering/README.md#add-easyquery-to-your-own-project) |
| Data filtering (MVC) | The controller actions and two views | [Mvc.DataFiltering](Mvc.DataFiltering/README.md#add-easyquery-to-your-own-project), [Mvc.DataFiltering.Auth](Mvc.DataFiltering.Auth/README.md#add-easyquery-to-your-own-project) (queries per user) |

For the details behind each step, see [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project)
and [Getting started for ASP.NET Core + MVC or Razor Pages](https://korzh.com/easyquery/docs/getting-started/asp-net-core/aspnet-core-razor)
in the documentation.

## Other platforms

* [ASP.NET Core + Angular](https://github.com/easyquery/eq-samples-aspnetcore-angular) — 2 samples
* [ASP.NET Core + Blazor](https://github.com/easyquery/eq-samples-aspnetcore-blazor) — 2 samples
* [ASP.NET Core + React](https://github.com/easyquery/eq-samples-aspnetcore-react) — 1 sample
* [ASP.NET Core + Vue](https://github.com/easyquery/eq-samples-aspnetcore-vue) — 2 samples
* [ASP.NET Core + Stencil](https://github.com/easyquery/eq-samples-aspnetcore-stencil) — 1 sample
* [ASP.NET MVC 5 (.NET Framework)](https://github.com/easyquery/eq-samples-aspnet4-mvc) — 5 samples
* [ASP.NET Web Forms (.NET Framework)](https://github.com/easyquery/eq-samples-aspnet4-webforms) — 2 samples
* [Windows Forms](https://github.com/easyquery/eq-samples-winforms) — 3 samples
* [WPF](https://github.com/easyquery/eq-samples-wpf) — 2 samples

## Links

* [EasyQuery home page](https://korzh.com/easyquery)
* [Documentation](https://korzh.com/easyquery/docs)
* [Live demo](https://korzh.com/demo)
* [All EasyQuery samples](https://github.com/easyquery/DotNetSamples)
