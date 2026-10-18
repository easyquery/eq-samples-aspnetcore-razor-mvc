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

1. Get the code: `git clone https://github.com/easyquery/eq-samples-aspnetcore-razor-mvc.git`, or **Code → Download ZIP** on this page.
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

EasyQuery needs a license key. A free 30-day trial key takes about a minute to get:
[register on korzh.com](https://korzh.com/easyquery?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc#get-started).

Put it into `appsettings.json` of the app that serves the EasyQuery API:

```json
"EasyQuery": {
  "LicenseKey": "your trial key goes here"
}
```

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
