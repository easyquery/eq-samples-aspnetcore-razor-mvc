# Razor.AdvancedSearch

An ASP.NET Core Razor Pages sample that implements the **advanced search scenario**: a full-page query builder where users pick entities and columns, define conditions, run the query and work with the result — with a responsive layout that adapts to small screens.

What you will see on the page:

* **Entities** panel — the tree of all entities and attributes of the data model; drag or click to add them to the query.
* **Columns** panel — the result columns (with sorting and aggregate functions).
* **Conditions** panel — the query conditions, grouped with AND/OR.
* **Result** panel — the result grid and a chart (Chart.js). The grid supports **server-side multi-column sorting** (click a header), **column resizing** with widths remembered in `localStorage`, and paging (30 rows).
* **Full-text search** in the result header — the text is sent to the server and turned into a group of hidden conditions (see `UseQueryTuner` in `Services/EasyQueryConfigurator.cs`). Use `||` to search for several terms.
* **Export** — PDF, Excel and CSV, plus **Export Async** (Excel) that runs as a background job for large results.
* **Toolbar / sidebar** — New, Load, Clear, Save, Save As, Remove and Fetch Data. A set of predefined queries is included (`App_Data/dm-nwind/queries`).

All EasyQuery settings are in one class — `Services/EasyQueryConfigurator.cs` (an `IEasyQueryConfigurator` registered with `AddEasyQuery<EasyQueryConfigurator>()`); `app.MapEasyQuery(...)` in `Program.cs` is left empty and uses the default endpoint `/api/easyquery`.

## Tech info

| | |
|---|---|
| .NET | **.NET 10** (`net10.0`) |
| Platform | ASP.NET Core |
| UI framework | **Razor Pages** |
| EasyQuery for .NET | **7.5.0** — `Korzh.EasyQuery.AspNetCore`, `Korzh.EasyQuery.EntityFrameworkCore.Relational`, `Korzh.EasyQuery.DataExport`, `Korzh.EasyQuery.RazorUI`, `Korzh.EasyQuery.SqLiteGate`, `Korzh.EasyQuery.SqlServerGate` |
| EasyData exporters | 1.6.0 — `EasyData.Exporters.PdfSharp` (PDF), `EasyData.Exporters.ClosedXML` (Excel) |
| EasyQuery.JS | **7.5.1**, loaded from CDN (`eq.enterprise.min.js`, `eq.core.min.css`, `eq.view.min.css`); charts by Chart.js |
| Query engine | SQL (`EasyQueryManagerSql`) |
| Data model source | EF Core `DbContext` (`UseDbContext<AppDbContext>`); an XML model file (`App_Data/NWindSQL.xml`) is included as an alternative |
| EasyQuery API | Middleware — `app.MapEasyQuery()`, endpoint `/api/easyquery`, configured by `EasyQueryConfigurator` |
| Database | SQLite through EF Core 10 (SQL Server is prepared in commented code) |
| Query storage | `FileQueryStore` (XML files in `App_Data`), or `SessionQueryStore` — see below |

## Project structure

| File / folder | What's inside |
|---|---|
| `Program.cs` | Services (DbContext, session, EasyQuery + exporters + background jobs), `MapEasyQuery`, demo DB initialization |
| `Services/EasyQueryConfigurator.cs` | **All EasyQuery settings**: manager, model loader, query store, model tuner, full-text search query tuner, SQL formats |
| `Services/SessionQueryStore.cs` | A query store that keeps queries in the user's session, seeded from the files in `App_Data` |
| `CustomEasyQueryManagerSql.cs` | An (unused) manager subclass showing where to modify SQL before execution |
| `ValuesController.cs` | An example API (`/api/values/list1`) returning a custom value list |
| `Pages/Index.cshtml` | The page and EasyQuery.JS initialization (`AdvancedSearchView`) |
| `Pages/Shared/_Layout.cshtml` | The app bar with query commands and the sidebar |
| `Models/NWind/`, `Data/AppDbContext.cs` | Entity classes and the DbContext of the demo (Northwind) database |
| `Data/DbInitializeExtensions.cs` | Creates and seeds the database on the first run |
| `App_Data/dm-nwind/queries/` | Predefined queries for the `nwind` model |
| `App_Data/NWindSQL.xml` | A data model saved to an XML file (not used by default) |
| `App_Data/EqDemoData.zip` | Demo data used to seed the database |

## How to start

1. Install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0).
2. Run the project:

   ```bash
   dotnet run
   ```

   or open `../Samples.AspNetCore.sln` and start `EqDemo.AspNetCoreRazor.AdvancedSearch`.
3. Open https://localhost:5001 (or http://localhost:5000).

You don't need a license key to start. On the first run the page shows a **No license** dialog:
enter your email, then the code we send to it, and you get a free 30-day trial key. The key is
saved to `EasyQuery.lck`; restart the sample to apply it. Already a customer? Put your key into
user secrets (run in the project folder) and the dialog won't appear:

```bash
dotnet user-secrets set "EasyQuery:LicenseKey" "<your key>"
```

or into `"EasyQuery": { "LicenseKey": "<your key>" }` in `appsettings.json`. See
[License key](../README.md#license-key) for more.

On the first run the application creates the SQLite database `eqdemo-sqlite.db` in the project folder and fills it with the demo data from `App_Data/EqDemoData.zip`. To reset the data, stop the app and delete the `eqdemo-sqlite.db*` files.

Try it: click **Load Query** and open one of the predefined queries, or build your own — add a few columns from the Entities panel, add a condition, press **Fetch Data**. Then click a column header to sort, or type something in the search box above the result.

## Next steps

### Connect the sample to your database

The sample builds its data model from `AppDbContext` — every `DbSet` becomes an entity in the
Entities panel — and runs the queries as SQL through the same EF Core connection. To show your
database instead of Northwind:

1. **Connection string.** `appsettings.json` → `ConnectionStrings:EqDemoDbLite`. The key is read in
   `Program.cs` and in `Services/EasyQueryConfigurator.cs`: put your connection string under it, or
   rename it in both places.
2. **EF Core provider.** In `Program.cs`, replace `options.UseSqlite(dbConnectionString)` with the
   provider for your database. For SQL Server it's the commented `options.UseSqlServer(...)` line,
   and the package is already referenced. For another database, add its EF Core 10 provider
   package (e.g. `Npgsql.EntityFrameworkCore.PostgreSQL`).
3. **SQL dialect.** In `EasyQueryConfigurator.cs`, remove the `options.UseSqlFormats(FormatType.Sqlite, ...)`
   call: without it, the generated SQL follows the provider of the `DbContext`. (Or keep it and
   name your database: `FormatType.MsSqlServer`, `FormatType.PostgreSql`, `FormatType.MySql`, …)

   The `.RegisterDbGate<...SqLiteGate>()` line in `Program.cs` can stay: a DB gate isn't used while
   the model comes from a `DbContext`. It's needed only to [load the model from the database schema](#load-the-model-directly-from-the-database-schema),
   and then it must be the gate for your database: `SqlServerGate`, `NpgSqlGate`, `MySqlGate`,
   `OdpCoreGate` (Oracle), each in a `Korzh.EasyQuery.<gate name>` package.
4. **Your entities.** Replace the Northwind classes in `Models/NWind/` and the `DbSet`s in
   `Data/AppDbContext.cs` with yours. If you don't have the classes yet, generate them from the
   database. The project folder has a tool manifest for `dotnet-ef`, so it's two commands to
   prepare, and then the scaffolding itself (the example is for SQL Server):

   ```bash
   dotnet tool restore
   dotnet add package Microsoft.EntityFrameworkCore.Design
   dotnet ef dbcontext scaffold "<your connection string>" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models/MyDb --namespace EqDemo.Models --context AppDbContext --context-dir Data --context-namespace EqDemo --no-onconfiguring --force
   ```

   The last command overwrites `Data/AppDbContext.cs` with a context for your tables. Then delete
   `Models/NWind/`. (`dotnet ef` warns that the tool is older than the runtime; the scaffolding
   works anyway.)
5. **Demo seeding.** Remove `app.EnsureDbInitialized(dbConnectionString, app.Environment);` from
   `Program.cs`. After that `Data/DbInitializeExtensions.cs`, `App_Data/EqDemoData.zip` and the
   `Korzh.DbUtils.*` packages are no longer used and can go too.
6. **Northwind leftovers.** In `EasyQueryConfigurator.cs`, remove the `options.UseModelTuner(...)`
   block: it looks for `Order.ShipRegion` and other Northwind attributes and fails on a model
   without them. Delete the predefined queries in `App_Data/dm-nwind/queries`.

Start the app: the Entities panel now lists your tables.

### Add EasyQuery to your own project

These steps bring the advanced search page into an existing ASP.NET Core application (.NET 8 or
later) that works with its database through EF Core.

1. **Install the packages.**

   ```bash
   dotnet add package Korzh.EasyQuery.AspNetCore
   dotnet add package Korzh.EasyQuery.EntityFrameworkCore.Relational
   dotnet add package Korzh.EasyQuery.DataExport
   dotnet add package EasyData.Exporters.PdfSharp
   dotnet add package EasyData.Exporters.ClosedXML
   ```

   The two `EasyData.Exporters.*` packages add PDF and Excel export; leave them out if you don't
   need it. The sample's versions are in [`Directory.Packages.props`](../Directory.Packages.props).
   Keep the EasyQuery.JS version in the page's CDN links (`cdn.korzh.com/eq/7.5.1/...`) in line
   with the `Korzh.EasyQuery.*` packages.
2. **Add the server side.** Copy [`Services/EasyQueryConfigurator.cs`](Services/EasyQueryConfigurator.cs)
   into your project and adjust it:
   * the namespace, and `AppDbContext` in `options.UseDbContext<AppDbContext>()` → your `DbContext`;
   * `options.DefaultModelId = "nwind"` → a name of your own;
   * remove the two connection string lines (`GetConnectionString(...)` and
     `options.ConnectionString = ...`): with a `DbContext`, EasyQuery uses its connection;
   * remove the `UseModelTuner(...)` block and the `UseSqlFormats(...)` call;
   * remove the `session` branch of the query store, unless you copy `Services/SessionQueryStore.cs`
     too.

   Then register it in `Program.cs`:

   ```csharp
   using EasyData.Export;
   using Korzh.EasyQuery.Services;
   ...
   builder.Services.AddDistributedMemoryCache();
   builder.Services.AddSession();

   builder.Services.AddEasyQuery<EasyQueryConfigurator>()
       .UseSqlManager()
       .AddDefaultExporters()
       .AddDataExporter<PdfDataExporter>("pdf")
       .AddDataExporter<ExcelDataExporter>("excel")
       .AddBackgroundJobsService()        // for "Export Async"
       .UseSessionCache();

   // the PDF exporter needs the non-Unicode code pages
   System.Text.Encoding.RegisterProvider(System.Text.CodePagesEncodingProvider.Instance);
   ...
   app.UseRouting();
   app.UseSession();
   ...
   app.MapEasyQuery();                    // the EasyQuery API at /api/easyquery
   ```

3. **Add the UI.** Copy the page and what it depends on:
   * `Pages/Index.cshtml` → e.g. `Pages/QueryBuilder.cshtml`, with a page model of its own
     (`public class QueryBuilderModel : PageModel { }`, and `@model QueryBuilderModel` in the page);
   * `Pages/Shared/_Layout.cshtml` → `Pages/Shared/_EasyQueryLayout.cshtml`, and set
     `Layout = "_EasyQueryLayout";` in the page. The app bar and the sidebar with the query
     commands (New, Load, Save, Fetch Data…) are in this layout, and the view finds them by ID;
   * `wwwroot/css/easyquery.css`, `layout.css`, `index.css`, `wwwroot/js/index.js` and
     `wwwroot/images/`.

   In the page script, remove the `.setAppName(...)` and `.useBackend(...)` lines: they tell the
   library that it runs in this sample.

   For a first look without copying anything, add the `Korzh.EasyQuery.RazorUI` package: it
   brings a ready-made, simpler query page that opens at `/EasyQuery/AdvancedSearch` inside your
   own `_Layout`. Its settings are the static properties of `Korzh.EasyQuery.RazorUI.Pages.AdvancedSearch`
   (`ExportFormats`, `ShowSqlPanel`, …), set in `Program.cs`.
4. **Connect your data.** `options.UseDbContext<...>()` from step 2 is all it takes: every `DbSet`
   of your context becomes an entity, and the queries run through the context's connection, in
   the SQL of its provider. Hide what users shouldn't see with `[EqEntity(false)]` on a class or
   `[EqEntityAttr(false)]` on a property, or with a filter:
   `options.UseDbContext<MyDbContext>(loaderOptions => loaderOptions.AddFilter(entity => ...))`.
5. **Choose where queries are stored.** The configurator uses `FileQueryStore`: XML files in
   `App_Data/dm-<model ID>/queries` (the folder is created on the first save).
6. **Set the license key**, as described in [License key](../README.md#license-key). Until there
   is one, the page shows the **No license** dialog instead of the query builder.

More in the docs: [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project).

### Change the sample

Almost everything below is changed in `Services/EasyQueryConfigurator.cs`.

#### Keep queries per browser session instead of shared files (configuration)

By default queries are saved as XML files in `App_Data/dm-nwind/queries`, shared by everyone. Set `QueryStoreMode` to `session` and each browser session gets its own copy of the predefined queries (changes are lost when the session ends — good for a public demo):

```json
"QueryStoreMode": "session"
```

in `appsettings.json`, or as an environment variable — `Program.cs` reads the variables with the `AdvancedSearch_` prefix:

```bash
AdvancedSearch_QueryStoreMode=session
```

To store the files as JSON instead of XML, change `FileFormat = "xml"` to `"json"` in the `FileQueryStoreSettings`.

#### Move the demo database to SQL Server (uncomment + change)

To keep the Northwind data but have it in SQL Server, do steps 1–3 of
[Connect the sample to your database](#connect-the-sample-to-your-database) (the `_alt_EqDemoDb`
entry in `appsettings.json` has an example connection string), and leave the seeding in place:
in `Data/DbInitializeExtensions.cs`, replace `options.UseSqlite(connectionString)` with the
commented `options.UseSqlServer(...)`. The demo data is then created in the new database on the
first start.

#### Load the model from the XML file

`App_Data/NWindSQL.xml` is a data model saved to a file (it's copied to the output folder by the `.csproj`). To use it instead of the DbContext, replace `options.UseDbContext<AppDbContext>();` with:

```csharp
using Microsoft.Data.Sqlite;
...
options.DefaultModelId = "NWindSQL";                                  // FileModelLoader looks for App_Data/<model ID>.xml or .json
options.UseModelLoader(_ => new FileModelLoader("App_Data"));
options.UseDbConnection<SqliteConnection>();                          // uses options.ConnectionString set above
```

Things to adjust together with it:

* The model ID becomes the folder name for saved queries — rename `App_Data/dm-nwind` to `App_Data/dm-NWindSQL` (or rename the XML file to `nwind.xml` and keep `DefaultModelId = "nwind"`).
* Saved queries and the model tuner (`UseModelTuner`) refer to entities and attributes by their IDs (`Order.ShipRegion`, `Category.CategoryName`, `Product.Category`). If the file model uses different IDs, `FindEntityAttr` returns `null` — update or remove those lines.

#### Load the model directly from the database schema

The SQLite gate is already registered in `Program.cs` (`.RegisterDbGate<SqLiteGate>()`), so you only need to replace `options.UseDbContext<AppDbContext>();` with:

```csharp
options.UseDbConnection<SqliteConnection>();
options.UseDbConnectionModelLoader(loaderOptions => {
    loaderOptions.IgnoreViews();
    // loaderOptions.AddTableFilter(...), UseSchemas("dbo"), DoNotPrettifyNames(), ...
});
```

The same note about `UseModelTuner` and the saved queries applies: the IDs in a schema-based model may differ.

#### Add hidden conditions to every query

The query tuner (`options.UseQueryTuner(...)`) runs for each request and already adds the full-text search conditions to `query.ExtraConditions`. Add your own conditions there — after `Conditions.Clear()` — to restrict the data for everyone, invisibly to users:

```csharp
options.UseQueryTuner(manager => {
    var query = manager.Query;
    query.ExtraConditions.Conditions.Clear();

    query.ExtraConditions.AddSimpleCondition("Order.ShipCountry", "Equal", "USA");   // always applied

    // ...the full-text search part stays below
});
```

#### Change the full-text search

In the same query tuner, `DbFullTextSearchOptions` controls what is searched:

* `IncludeDateTimeFields`, `IncludeNumericFields` — search in dates and numbers too (both `true` in the sample);
* `Entities` / `IncludeRootEntity` — the default scope when the query has no columns or conditions.

The search scope is otherwise taken from the query itself: its result columns, or the entities used in its conditions.

#### Fill the "Ship Region" list

The model tuner gives `Order.ShipRegion` a custom list editor (`CustomListValueEditor("Lookup", "Lookup")`), but nothing provides values for the `Lookup` list yet, so it's empty. Provide them on the server:

```csharp
options.AddValueListResolver(request => {
    if (request.ListName == "Lookup") {
        return new List<ListItem> {
            new ListItem("WA", "Washington"),
            new ListItem("OR", "Oregon")
        };
    }
    return null;   // not our list - let other resolvers handle it
});
```

or on the client, with the `onListRequest` handler in `Pages/Index.cshtml` — e.g. from the `/api/values/list1` endpoint of `ValuesController`:

```js
onListRequest: function (params, onResult) {
    if (params.editorId === 'Lookup') {
        fetch('/api/values/list1').then(r => r.json()).then(onResult);
        return true;    // handled here, don't ask the EasyQuery API
    }
    return false;
},
```

#### Modify SQL before execution

`CustomEasyQueryManagerSql.cs` overrides `PrepareDbCommand` — the place to adjust `statement.SQL` (add hints, rewrite table names, log it). It's not wired in; to use it, change the first line of `Configure`:

```csharp
options.UseManager<CustomEasyQueryManagerSql>();   // instead of UseManager<EasyQueryManagerSql>()
```

#### Change the page behavior

In `Pages/Index.cshtml`, `viewOptions` of the `AdvancedSearchView`:

* `loadQueryOnStart: true` plus `defaultQueryId: '<query id>'` — open the page with one of the saved queries.
* `const showPanelButtons = false` — set to `true` to always show the buttons in the Columns and Conditions panels.
* `widgets.queryPanel.minDate` / `maxDate` (commented) — limit the date pickers, e.g. `minDate: '2025-01-01', maxDate: 'today'`.
* `locale` (commented) and `localeSettings.shortDateFormat` — the date format and UI language.
* `widgets.resultGrid` — `sortable`, `allowColumnResize`, and `columnWidthPersistence` (`scope: 'query'` keeps separate column widths for each saved query).
* `result.showChart`, `result.paging.pageSize` — the chart and the page size.
* `serverExporters` and the export buttons in the markup — the export formats (`pdf` and `excel` must be registered with `.AddDataExporter<...>()` in `Program.cs`).

#### The built-in Advanced Search page and its settings (uncomment)

`Program.cs` sets `Korzh.EasyQuery.RazorUI.Pages.AdvancedSearch.ExportFormats` and has a commented `...ShowSqlPanel = true;` line. These static settings belong to the ready-made page that the `Korzh.EasyQuery.RazorUI` package adds to the app at `/EasyQuery/AdvancedSearch` — not to this sample's own `Pages/Index.cshtml`, which has its own markup. Uncomment the line and open `/EasyQuery/AdvancedSearch` to see the generated SQL under the result.

## Links

* [Live demo of advanced search](https://demo.korzh.com/easyquery-asp-net-core-razor/advanced-search?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* [All samples for ASP.NET Core MVC/Razor](../README.md)
* [All EasyQuery samples](https://github.com/easyquery)
* [EasyQuery home page](https://korzh.com/easyquery?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* [EasyReport Starter Kit](https://korzh.com/easy-report-starter-kit?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc): a complete ad-hoc reporting app built on EasyQuery
