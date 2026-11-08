# Razor.DataFiltering

An ASP.NET Core Razor Pages sample that implements the **data-filtering scenario** in the simplest possible way: an Orders list with an EasyQuery filter bar on top, where users build their own filtering conditions.

What you will see on the page:

* **FilterBar** — an EasyQuery widget where users add conditions like "Customer country is USA" or "Freight is greater than 100". Pressing **Apply** re-fetches the list.
* **Result grid** — EasyQuery.JS's own grid (20 rows per page, no chart), filled with the data returned by the server.

The whole server side is one call: `app.MapEasyQuery(...)` in `Program.cs` adds the EasyQuery API at `/data-filtering`, and `options.UseEntity(...)` builds the data model from the `Order` class and applies the filters to an EF Core `IQueryable<Order>`. There are no controllers and the page model is empty.

Compare with [Mvc.DataFiltering](../Mvc.DataFiltering/README.md), which does the same with an MVC controller and a server-rendered result list.

## Tech info

| | |
|---|---|
| .NET | .NET 10 (`net10.0`) |
| Platform | ASP.NET Core |
| UI framework | **Razor Pages** |
| EasyQuery for .NET | **7.5.0** — `Korzh.EasyQuery.AspNetCore`, `Korzh.EasyQuery.Linq` (also `Korzh.EasyQuery.Db`, `Korzh.EasyQuery.SqLiteGate` for the "load model from DB" option below) |
| EasyQuery.JS | **7.5.1**, loaded from CDN (`eq.enterprise.min.js`, `eq.core.min.css`) |
| Query engine | LINQ over EF Core |
| Data model source | The `Order` class and its navigation properties (`UseEntity`) |
| EasyQuery API | Middleware — `app.MapEasyQuery(...)`, endpoint `/data-filtering` |
| Database | SQLite through EF Core 8 (SQL Server is prepared in commented code) |

## Project structure

| File / folder | What's inside |
|---|---|
| `Program.cs` | Services, the `MapEasyQuery` middleware with all EasyQuery settings, demo DB initialization |
| `Pages/Index.cshtml` | The page: FilterBar + result grid, EasyQuery.JS initialization (`DataFilterView`) |
| `Models/NWind/` | Entity classes of the demo (Northwind) database |
| `Data/AppDbContext.cs` | EF Core DbContext |
| `Data/DbInitializeExtensions.cs` | Creates and seeds the database on the first run |
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

   or open `../Samples.AspNetCore.sln` and start `EqDemo.AspNetCoreRazor.DataFiltering`.
4. Open https://localhost:5001 (or http://localhost:5000).

On the first run the application creates the SQLite database `eqdemo-sqlite.db` in the project folder and fills it with the demo data from `App_Data/EqDemoData.zip`. To reset the data, stop the app and delete the `eqdemo-sqlite.db*` files.

## Next steps

### Connect the sample to your database

The sample filters one entity, `Order`, read through EF Core's `AppDbContext`. To filter a list
from your own database:

1. **Connection string.** `appsettings.json` → `ConnectionStrings:EqDemoDb`.
2. **EF Core provider.** In `Program.cs`, replace `options.UseSqlite(DbConnectionString)` with the
   provider for your database — for SQL Server, the commented `options.UseSqlServer(...)` line
   (the package is already referenced). For another database, add its EF Core 8 provider package.
3. **Your entities.** Replace the Northwind classes in `Models/NWind/` and the `DbSet`s in
   `Data/AppDbContext.cs` with yours. To generate them from the database (the project already
   references `Microsoft.EntityFrameworkCore.Tools`; the example is for SQL Server):

   ```bash
   dotnet tool install --global dotnet-ef
   dotnet ef dbcontext scaffold "<your connection string>" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models/MyDb --namespace EqDemo.Models --context AppDbContext --context-dir Data --context-namespace EqDemo --no-onconfiguring --force
   ```

   The scaffolding overwrites `Data/AppDbContext.cs`. Then delete `Models/NWind/`.
4. **The entity to filter.** In the `app.MapEasyQuery(...)` block of `Program.cs`, `options.UseEntity(...)`
   returns the list to filter. Replace `.Orders.Include(...)` with your `DbSet` and the related
   data to show in the filter bar:

   ```csharp
   options.UseEntity((manager) => manager.Services
           .GetRequiredService<AppDbContext>()
           .Books
           .Include(b => b.Author)
           .AsQueryable());
   ```

5. **Demo seeding.** Remove `app.EnsureDbInitialized(configuration, app.Environment);` from
   `Program.cs`. `Data/DbInitializeExtensions.cs`, `App_Data/EqDemoData.zip` and the
   `Korzh.DbUtils.*` packages can go with it.

Start the app: the filter bar offers the properties of your entity and the included ones, and the
grid shows its records.

### Add EasyQuery to your own project

These steps add a filter bar to a list page of an existing ASP.NET Core Razor Pages application
(.NET 8 or later) that reads the list through EF Core.

1. **Install the packages.**

   ```bash
   dotnet add package Korzh.EasyQuery.AspNetCore
   dotnet add package Korzh.EasyQuery.Linq
   ```

   The sample's versions are in [`Directory.Packages.props`](../Directory.Packages.props). Keep the
   EasyQuery.JS version in the page's CDN links (`cdn.korzh.com/eq/7.5.1/...`) in line with them.
2. **Add the server side.** In `Program.cs`:

   ```csharp
   using Microsoft.EntityFrameworkCore;
   using Korzh.EasyQuery.Services;
   ...
   builder.Services.AddEasyQuery();
   ...
   app.MapEasyQuery(options => {
       options.Endpoint = "/data-filtering";
       options.UseEntity((manager) => manager.Services
               .GetRequiredService<MyDbContext>()
               .Books
               .Include(b => b.Author)
               .AsQueryable());
       options.BuildQueryOnSync = true;
       options.SaveNewQuery = false;
   });
   ```

   That's the whole server side: the EasyQuery API at `/data-filtering` builds the data model from
   the entity type and applies the user's conditions to the `IQueryable` you return.
3. **Add the UI.** From [`Pages/Index.cshtml`](Pages/Index.cshtml), copy into your page:
   * the two placeholders: `<div id="FilterBar"></div>` and `<div id="ResultPanel"></div>`;
   * the stylesheet `https://cdn.korzh.com/eq/7.5.1/eq.core.min.css`. The sample puts it into a
     `Styles` section, which the layouts of the standard templates don't render: either move the
     `<link>` out of the section, or add `@await RenderSectionAsync("Styles", required: false)`
     to the `<head>` of your `_Layout.cshtml`;
   * the `Scripts` section: the `eq.enterprise.min.js` script and the initialization script.
     Remove the `.setAppName(...)` and `.useBackend(...)` lines from it — they tell the library that
     it runs in this sample — and keep `.useEndpoint(...)` in line with `options.Endpoint`.
4. **Connect your data.** That's the `UseEntity(...)` lambda from step 2. Captions and visibility
   come from attributes on your classes — see [Filter a different entity](#filter-a-different-entity-or-include-more-related-data).
5. **Query storage.** The sample doesn't save filters; [Save filters and open the page with a predefined one](#save-filters-and-open-the-page-with-a-predefined-one)
   shows how to add it.
6. **Set the license key**, as described in [License key](../README.md#license-key). Until there
   is one, the page shows the **No license** dialog instead of the filter bar.

More in the docs: [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project).

### Change the sample

#### Move the demo database to SQL Server (uncomment)

1. `appsettings.json` — comment the SQLite `EqDemoDb` connection string and uncomment the SQL Server one (or put your own).
2. `Program.cs` — replace `options.UseSqlite(DbConnectionString)` with the commented `options.UseSqlServer(...)` line.
3. `Data/DbInitializeExtensions.cs` — do the same inside `DbInitializer.Create(...)`.

#### Give the model a fixed ID (uncomment)

`//options.DefaultModelId = "nwind";` in the `MapEasyQuery` block. A stable model ID matters as soon as you store queries (file query stores use it as a folder name — `App_Data/dm-nwind/queries`) or cache the model.

#### Filter a different entity or include more related data

The `UseEntity` lambda defines both the data model and the data source. Every `Include`d navigation property becomes available in the filter bar:

```csharp
app.MapEasyQuery(options => {
    options.Endpoint = "/data-filtering";
    options.UseEntity(manager => manager.Services
            .GetRequiredService<AppDbContext>()
            .Orders
            .Include(o => o.Customer)
            .Include(o => o.Employee)
            .Include(o => o.Items)              // add order lines to the model
            .AsQueryable(),
        loaderOptions => loaderOptions.SortAlpabetically = false);
    ...
});
```

To filter another list, return a different `IQueryable` (e.g. `.Customers`). The captions and visibility of properties come from the attributes on the model classes: `[Display(Name = ...)]`, `[EqListValueEditor]` (a list of values instead of a text box — see `Employee.Country`), `[EqEntityAttr(UseInConditions = false)]` to hide a property from the filter.

#### Add a fixed (hidden) filter

Whatever you put into the `IQueryable` returned by `UseEntity` is applied before the user's conditions and is never shown in the UI:

```csharp
options.UseEntity(manager => manager.Services
        .GetRequiredService<AppDbContext>()
        .Orders
        .Include(o => o.Customer)
        .Include(o => o.Employee)
        .Where(o => o.ShipCountry == "USA")     // always applied
        .AsQueryable());
```

#### Save filters and open the page with a predefined one

The sample doesn't store queries. To add Save/Load and a default filter:

1. Register a query store in the `MapEasyQuery` block and let new queries be saved:

   ```csharp
   options.DefaultModelId = "nwind";
   options.SaveNewQuery = true;
   options.UseQueryStore(_ => new FileQueryStore("App_Data"));
   ```

2. In `Pages/Index.cshtml` load the query on start:

   ```js
   var viewOptions = {
       loadModelOnStart: true,
       loadQueryOnStart: true,
       defaultQueryId: 'default-filter',   // App_Data/dm-nwind/queries/default-filter.json
       ...
   };
   ```

   Save/Load buttons can be added the same way as in `Views/Order/Orders.cshtml` of the [Mvc.DataFiltering](../Mvc.DataFiltering/README.md) sample (`context.saveQuery(...)` / `context.loadQuery(...)`).

#### Change the result grid

In `Pages/Index.cshtml`, the `result` section of `viewOptions`:

```js
result: {
    showChart: false,       // true - show a chart next to the grid
    paging: {
        pageSize: 20
    }
}
```

#### Load the model from the database schema instead of a class

The project already references `Korzh.EasyQuery.Db` and `Korzh.EasyQuery.SqLiteGate`, and `Program.cs` already imports `Microsoft.Data.Sqlite`. To build the model by scanning the DB meta-data (all tables, not only `Order`), switch to the SQL manager:

```csharp
builder.Services.AddEasyQuery()
    .UseSqlManager()
    .RegisterDbGate<Korzh.EasyQuery.DbGates.SqLiteGate>();   // the gate for your DB type

...

app.MapEasyQuery(options => {
    options.Endpoint = "/data-filtering";
    options.DefaultModelId = "nwind";
    options.UseDbConnection<SqliteConnection>(DbConnectionString);
    options.UseDbConnectionModelLoader();                      // replaces options.UseEntity(...)
    options.BuildQueryOnSync = true;
    options.SaveNewQuery = false;
});
```

Note that with the SQL manager a query needs result columns, and the data-filtering view has no UI for picking them — load a predefined query with columns on start (see above), or use the Advanced Search view as in the [Razor.AdvancedSearch](../Razor.AdvancedSearch/README.md) sample, which is built around a SQL model.

## Links

* [Live demo of data filtering](https://demo.korzh.com/easyquery-asp-net-core-razor/data-filtering?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* [All samples for ASP.NET Core MVC/Razor](../README.md)
* [All EasyQuery samples](https://github.com/easyquery)
* [EasyQuery home page](https://korzh.com/easyquery?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* [Get a complete reporting portal](https://korzh.com/easy-report-starter-kit?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc) with EasyReport Starter Kit
