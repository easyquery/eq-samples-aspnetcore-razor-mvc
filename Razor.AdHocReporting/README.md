# Razor.AdHocReporting

An ASP.NET Core Razor Pages sample that implements the **ad-hoc reporting scenario**: logged-in users get a list of their own reports and can create, change, save and remove them — choosing columns, aggregations and conditions — then see the result as a grid and a chart and export it to PDF, Excel or CSV.

What you will see on the page (after logging in):

* **Reports** (left) — the current user's reports. The ⚙ menu has *New report*, *Save as...* and *Remove report*.
* **Columns** / **Aggregations** / **Conditions** — EasyQuery widgets to define the report.
* **Result** — the result grid with totals, a chart (Google Charts) and export buttons.

Every change is saved immediately (`syncReportOnChange: true` on the client, `SaveQueryOnSync = true` on the server). Reports are stored in the database, one set per user (`Services/ReportStore.cs`). Each new user starts with a few ready-made reports copied from `App_Data/Seed/*.json`.

The data model is built from the EF Core `AppDbContext` (all Northwind entities; the Identity tables and the `Report` entity itself are excluded), and reports are executed as SQL.

A variant of this sample with the client code written in TypeScript and bundled from npm packages is [RazorTypeScript.AdHocReporting](../RazorTypeScript.AdHocReporting/README.md).

## Tech info

| | |
|---|---|
| .NET | .NET 10 (`net10.0`) |
| Platform | ASP.NET Core |
| UI framework | **Razor Pages** |
| EasyQuery for .NET | **7.5.0** — `Korzh.EasyQuery.AspNetCore`, `Korzh.EasyQuery.EntityFrameworkCore.Relational`, `Korzh.EasyQuery.EntityFrameworkCore.Identity`, `Korzh.EasyQuery.DataExport`, `Korzh.EasyQuery.RazorUi`, `Korzh.EasyQuery.SqLiteGate`, `Korzh.EasyQuery.SqlServerGate` |
| EasyData exporters | 1.6.0 — `EasyData.Exporters.PdfSharp` (PDF), `EasyData.Exporters.ClosedXML` (Excel) |
| EasyQuery.JS | **7.5.1**, loaded from CDN (`eq.enterprise.min.js`, `eq.core.min.css`); charts by Google Charts |
| Query engine | SQL (`UseSqlManager()`) |
| Data model source | EF Core `DbContext` (`UseDbContextWithoutIdentity<AppDbContext>`) |
| EasyQuery API | Middleware — `app.MapEasyQuery(...)`, endpoint `/api/adhoc-reporting` |
| Authentication | ASP.NET Core Identity; report management is limited to the `eq-manager` role |
| Database | SQLite through EF Core 8 (SQL Server is prepared in commented code) |
| Report storage | `ReportStore` — the `Reports` table, per user |

## Project structure

| File / folder | What's inside |
|---|---|
| `Program.cs` | Services (DbContext, Identity, EasyQuery + exporters), the `MapEasyQuery` middleware with all EasyQuery settings, demo DB initialization |
| `Pages/Index.cshtml` | The report page and EasyQuery.JS initialization (`ReportView`). `[Authorize]` in `Index.cshtml.cs` |
| `Services/ReportStore.cs` | `IQueryStore` implementation that keeps reports in the `Reports` table, scoped to the current user |
| `Services/DefaultReportGenerator.cs` | Copies the seed reports from `App_Data/Seed` to a new user |
| `Models/Report.cs` | The entity for a stored report |
| `Models/NWind/` | Entity classes of the demo (Northwind) database |
| `Data/AppDbContext.cs` | The DbContext; inherits `IdentityDbContext` |
| `Data/DbInitializeExtensions.cs` | On start: creates/seeds the DB, adds the `eq-manager` role and the default user |
| `Areas/Identity/` | Login, Register and Logout pages. Registration adds the user to `eq-manager` and generates the seed reports |
| `App_Data/Seed/` | Report definitions (JSON) given to each new user |
| `App_Data/EqDemoData.zip` | Demo data used to seed the database |

## How to start

1. Install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0) (or later).
2. Put your EasyQuery license key into `appsettings.json`:

   ```json
   "EasyQuery": {
     "LicenseKey": "<your key>"
   }
   ```

   or, better, into user secrets (run in the project folder):

   ```bash
   dotnet user-secrets set "EasyQuery:LicenseKey" "<your key>"
   ```

   You can get a trial key in the [Korzh.com Client's Area](https://account.korzh.com) after registering an account.

3. Run the project:

   ```bash
   dotnet run
   ```

   or open `../Samples.AspNetCore.sln` and start `EqDemo.AspNetCoreRazor.AdhocReporting`.
4. Open https://localhost:5001 (or http://localhost:5000) and log in with the default user:

   * login: **demo@korzh.com**
   * password: **demo**

   or register a new account.

On the first run the application creates the SQLite database `eqdemo-sqlite.db`, seeds it with the demo data, and creates the default user with a set of reports. To reset everything, stop the app and delete the `eqdemo-sqlite.db*` files. To reset only the demo user and its reports, set `"resetDefaultUser": true` in `appsettings.json` for one start.

## Next steps

### Connect the sample to your database

`AppDbContext` holds three things at once: the Northwind tables, the users and roles (it's an
`IdentityDbContext`), and the users' reports (`Reports`). The data model is built from it without
the Identity tables and without `Report`. For your data, the simplest way is a second `DbContext`
for your database, while users and reports stay in the sample's own SQLite file:

1. **Connection string.** Add one for your database to `ConnectionStrings` in `appsettings.json`,
   e.g. `"MyDb": "..."`. Keep `EqDemoDb`: it's now the database of users and reports.
2. **Your `DbContext`.** Generate it with its entity classes from your database. Build the project
   once first; the project already references `Microsoft.EntityFrameworkCore.Tools`, and the
   example is for SQL Server:

   ```bash
   dotnet tool install --global dotnet-ef
   dotnet ef dbcontext scaffold "<your connection string>" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models/MyDb --namespace EqDemo.Models.MyDb --context MyDataContext --context-dir Data --context-namespace EqDemo --no-onconfiguring
   ```

   Then register it in `Program.cs`, next to `AppDbContext`:

   ```csharp
   builder.Services.AddDbContext<MyDataContext>(options =>
       options.UseSqlServer(builder.Configuration.GetConnectionString("MyDb")));
   ```

   For another database, add its EF Core 8 provider package and use its `Use...()` method.
3. **Build the model from it.** In the `app.MapEasyQuery(...)` block of `Program.cs`, replace
   `options.UseDbContextWithoutIdentity<AppDbContext>(...)` (with its filter) by:

   ```csharp
   options.UseDbContext<MyDataContext>();
   ```

   Reports are executed as SQL through this context's connection, in the SQL of its provider.
4. **Remove Northwind from `AppDbContext`.** Delete the `NWind` region and the `OrderDetail` lines
   of `OnModelCreating` in `Data/AppDbContext.cs`, then the `Models/NWind/` folder. Keep the
   `Reports` set.
5. **Demo seeding.** In `Data/DbInitializeExtensions.cs`, replace everything from the seeding-marker
   comment down to the end of the `if (context.Database.EnsureCreated()) { ... }` block with a single
   `context.Database.EnsureCreated();`. The role and the default user are still created. Delete the
   old `eqdemo-sqlite.db*` files, if any, so the database is created again without the Northwind
   tables.
6. **Seed reports.** Every new user gets copies of the reports in `App_Data/Seed/*.json`, and those
   are Northwind reports. Delete the files but keep the folder: the code reads it on every
   registration. Later, put reports of your own there (see [Change the seed reports](#change-the-seed-reports-and-the-default-user)).

Start the app and log in: the report designer offers your tables.

### Add EasyQuery to your own project

These steps bring ad-hoc reporting into an existing ASP.NET Core Razor Pages application (.NET 8
or later) that has users (ASP.NET Core Identity) and reads its data through EF Core. Reports
belong to users, so the page needs a logged-in user.

1. **Install the packages.**

   ```bash
   dotnet add package Korzh.EasyQuery.AspNetCore
   dotnet add package Korzh.EasyQuery.EntityFrameworkCore.Relational
   dotnet add package Korzh.EasyQuery.DataExport
   dotnet add package EasyData.Exporters.PdfSharp
   dotnet add package EasyData.Exporters.ClosedXML
   ```

   The two `EasyData.Exporters.*` packages add PDF and Excel export. If your data is in the same
   `DbContext` as Identity, add `Korzh.EasyQuery.EntityFrameworkCore.Identity` too (see step 4). The
   sample's versions are in [`Directory.Packages.props`](../Directory.Packages.props). Keep the
   EasyQuery.JS version in the page's CDN links (`cdn.korzh.com/eq/7.5.1/...`) in line with them.
2. **Add the server side.** In `Program.cs`:

   ```csharp
   using EasyData.Export;
   using Korzh.EasyQuery.Services;
   ...
   builder.Services.AddDistributedMemoryCache();
   builder.Services.AddSession();

   builder.Services.AddEasyQuery()
       .AddDefaultExporters()
       .AddDataExporter<PdfDataExporter>("pdf")
       .AddDataExporter<ExcelDataExporter>("excel")
       .UseSessionCache()
       .UseSqlManager();

   // the PDF exporter needs the non-Unicode code pages
   System.Text.Encoding.RegisterProvider(System.Text.CodePagesEncodingProvider.Instance);
   ...
   app.UseRouting();
   app.UseSession();
   app.UseAuthentication();
   app.UseAuthorization();

   app.MapEasyQuery(options => {
       options.DefaultModelId = "adhoc-reporting";
       options.Endpoint = "/api/adhoc-reporting";
       options.SaveNewQuery = true;
       options.SaveQueryOnSync = true;
       options.StoreModelInCache = true;
       options.StoreQueryInCache = true;

       options.UseDbContext<MyDataContext>();
       options.UseQueryStore(manager => new ReportStore(manager.Services));

       options.UseDefaultAuthProvider(provider => {
           // New, Save and Remove need the "eq-manager" role by default; this opens them to every logged-in user
           provider.RequireAuthorization(EqAction.NewQuery, EqAction.SaveQuery, EqAction.RemoveQuery);
       });
   });
   ```

   The CORS policy and the URL rewriting in the sample's `Program.cs` belong to the demo, not to
   EasyQuery: leave them out.
3. **Add the report storage.** Copy [`Models/Report.cs`](Models/Report.cs) and
   [`Services/ReportStore.cs`](Services/ReportStore.cs), change their namespaces, and in
   `ReportStore` replace `AppDbContext` with the `DbContext` of your users. Add the reports to that
   context and create the table:

   ```csharp
   public DbSet<Report> Reports { get; set; }
   ```

   ```bash
   dotnet ef migrations add AddReports --context ApplicationDbContext
   dotnet ef database update --context ApplicationDbContext
   ```

   (`ApplicationDbContext` is the name the project templates give it. `--context` is needed as
   soon as the project has more than one `DbContext`.)

   Ready-made reports for new users (`Services/DefaultReportGenerator.cs`, `App_Data/Seed`) are
   optional; the sample calls the generator from its Register page.
4. **Connect your data.** `options.UseDbContext<MyDataContext>()` in step 2 builds the model from
   your data's `DbContext`. If that's the same context as Identity, use
   `options.UseDbContextWithoutIdentity<...>()` instead, and leave `Report` out as the sample does:
   `loaderOptions => loaderOptions.AddFilter(entity => entity.ClrType != typeof(Report))`.
5. **Add the UI.** Copy [`Pages/Index.cshtml`](Pages/Index.cshtml) as e.g. `Pages/Reports.cshtml`,
   with an `[Authorize]` page model (`public class ReportsModel : PageModel { }`, and
   `@model ReportsModel` in the page), and `wwwroot/css/adhoc-reporting.css` and `root.css`. Then:
   * the page puts its stylesheets into a `Styles` section, which the layouts of the standard
     templates don't render: add `@await RenderSectionAsync("Styles", required: false)` to the
     `<head>` of your `_Layout.cshtml`;
   * the ⚙ menu is a Bootstrap 4 dropdown (`data-toggle="dropdown"`). The current templates use
     Bootstrap 5: replace `data-toggle=`, `data-placement=` and `data-html=` with
     `data-bs-toggle=`, `data-bs-placement=` and `data-bs-html=` all over the page, the
     `$('[data-toggle="tooltip"]')` selector in the script included;
   * the page enables the menu only for users in the `eq-manager` role (`IsQueryManager`). With
     `RequireAuthorization(...)` from step 2, set `IsQueryManager` to `true` for every logged-in
     user;
   * in the script, remove the `.setAppName(...)` and `.useBackend(...)` lines — they tell the
     library that it runs in this sample.
6. **Set the license key**, as described in [License key](../README.md#license-key). Until there
   is one, the page shows the **No license** dialog instead of the reports.

More in the docs: [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project).

### Change the sample

#### Move the demo database to SQL Server (uncomment)

1. `appsettings.json` — comment the SQLite `EqDemoDb` connection string and uncomment the `(localdb)\\MSSQLLocalDB` one (or put your own).
2. `Program.cs` — replace `options.UseSqlite(...)` with the commented `options.UseSqlServer(...)` line.
3. `Data/DbInitializeExtensions.cs` — do the same inside `DbInitializer.Create(...)`.

The SQL format of the generated statements follows the DbContext provider automatically.

#### Let every user manage reports (uncomment)

By default only users in the `eq-manager` role can create, save and remove reports. In the `options.UseDefaultAuthProvider(...)` block of `Program.cs`:

```csharp
options.UseDefaultAuthProvider((provider) => {
    // any logged-in user can manage reports
    provider.RequireAuthorization(EqAction.NewQuery, EqAction.SaveQuery, EqAction.RemoveQuery);

    // or: only users with some particular role
    //provider.RequireRole("report-admin", EqAction.NewQuery, EqAction.SaveQuery, EqAction.RemoveQuery);
});
```

`Pages/Index.cshtml` also checks the role (`IsQueryManager`) to enable or disable the ⚙ menu items — keep that check in sync with the rule above.

#### Show each user only their own data (uncomment)

The `options.AddPreFetchTunerWithHttpContext(...)` block in `Program.cs` runs before every query execution. The two commented lines add a hidden condition to each report:

```csharp
options.AddPreFetchTunerWithHttpContext((manager, context) => {
    string userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    manager.Query.ExtraConditions.AddSimpleCondition("Employees.EmployeeID", "Equal", userId);
});
```

Add `using System.Security.Claims;` at the top of `Program.cs`. This is a pattern to adapt: the demo Identity users are not linked to Northwind employees, so replace the attribute ID and the value with something that makes sense in your model (tenant ID, department, etc.).

#### Load the model directly from the database schema (uncomment + change)

`Program.cs` has the commented `.RegisterDbGate<SqLiteGate>()` / `.RegisterDbGate<SqlServerGate>()` lines for this. To build the model from the DB meta-data instead of the DbContext:

```csharp
using Microsoft.Data.Sqlite;
using Korzh.EasyQuery.DbGates;
...
builder.Services.AddEasyQuery()
                .AddDefaultExporters()
                .AddDataExporter<PdfDataExporter>("pdf")
                .AddDataExporter<ExcelDataExporter>("excel")
                .UseSessionCache()
                .UseSqlManager()                // NB: remove the ';' that ends this line now
                .RegisterDbGate<SqLiteGate>();  // or SqlServerGate
...
app.MapEasyQuery(options => {
    ...
    // instead of options.UseDbContextWithoutIdentity<AppDbContext>(...):
    options.UseDbConnection<SqliteConnection>(builder.Configuration.GetConnectionString("EqDemoDb"));
    options.UseDbConnectionModelLoader(loaderOptions => {
        // skip the Identity and Reports tables, e.g.:
        // loaderOptions.AddTableFilter(table => ...);
    });
});
```

`DbConnectionModelLoaderOptions` also has `UseSchemas(...)`, `IgnoreViews()`, `AddFieldFilter(...)`, `DoNotPrettifyNames()`, and others. Keep in mind that entity and attribute IDs in a schema-based model differ from the DbContext-based one, so the seed reports in `App_Data/Seed` may not open against it.

#### Load the model from a saved file

A model saved to a JSON or XML file (e.g. created in the EasyQuery Data Model Editor) can be used instead:

```csharp
// instead of options.UseDbContextWithoutIdentity<AppDbContext>(...):
options.UseModelLoader(_ => new FileModelLoader("App_Data"));
```

`FileModelLoader` looks for `App_Data/<model ID>.json` (or `.xml`), where the model ID is `DefaultModelId` — `adhoc-reporting` here. The SQL manager then needs a connection: add `options.UseDbConnection<SqliteConnection>(...)` as in the previous section.

#### Tune the model built from DbContext

`UseDbContextWithoutIdentity` takes the same options as `UseDbContext`. The sample already uses one of them to exclude the `Report` entity:

```csharp
options.UseDbContextWithoutIdentity<AppDbContext>(loaderOptions => {
    loaderOptions.AddFilter(entity => entity.ClrType != typeof(Report)
                                   && entity.ClrType != typeof(Supplier));   // hide one more entity
    loaderOptions.SortAlphabetically = true;
});
```

Properties can also be hidden or renamed with attributes on the model classes: `[Display(Name = ...)]`, `[EqEntityAttr(UseInResult = false)]`, `[EqEntity(false)]`.

#### Change the seed reports and the default user

* `App_Data/Seed/*.json` — the reports every new user gets. Save a report in the UI, find it in the `Reports` table (`QueryJson` column), and put it here as a new file.
* `Data/DbInitializeExtensions.cs` — the default user's e-mail and password (`_defaultUserEmail`, `_defaultUserPassword`).
* Password rules — the `AddDefaultIdentity<IdentityUser>(...)` call in `Program.cs`.

#### Change the report page

In `Pages/Index.cshtml`, `viewOptions` of the `ReportView`:

* `serverExporters: ['pdf', 'excel', 'excel-html', 'csv']` — the export formats. `pdf` and `excel` must be registered on the server with `.AddDataExporter<...>()` in `Program.cs`.
* `result.showChart` — turn the chart off; `result.paging.pageSize` — rows per page.
* `syncReportOnChange` — set to `false` to save reports only on an explicit *Save*.

## Links

* [Live demo of ad-hoc reporting](https://demo.korzh.com/easyquery-asp-net-core-razor/adhoc-reporting?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* [All samples for ASP.NET Core MVC/Razor](../README.md)
* [All EasyQuery samples](https://github.com/easyquery)
* [EasyQuery home page](https://korzh.com/easyquery?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* Need a full reporting app? Try [EasyReport Starter Kit](https://korzh.com/easy-report-starter-kit?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
