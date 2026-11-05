# RazorTypeScript.AdHocReporting

The **ad-hoc reporting** sample with the client code written in **TypeScript**. Server-side it is the same application as [Razor.AdHocReporting](../Razor.AdHocReporting/README.md) — the difference is how EasyQuery.JS gets to the browser:

* **Razor.AdHocReporting** loads the ready-made EasyQuery.JS bundle from the CDN and initializes it in an inline `<script>`.
* **This sample** installs EasyQuery.JS as **npm packages** (`@easyquery/core`, `@easyquery/ui`, `@easyquery/enterprise`, `@easydata/core`, `@easydata/ui`), and the page script `ts/adhoc-reporting.ts` imports what it needs. Rollup bundles it into `wwwroot/js/adhoc-reporting.js` (plus the CSS into `wwwroot/css/adhoc-reporting.css`). This is the setup to use when your front-end already has a build pipeline, or you want typed access to the EasyQuery.JS API.

What the application does: logged-in users get a list of their own reports and can create, change, save and remove them — choosing columns, aggregations and conditions — then see the result as a grid with totals and a chart, and export it to PDF, Excel or CSV. Reports are stored in the database per user (`Services/ReportStore.cs`); each new user starts with the reports from `App_Data/Seed/*.json`.

## Tech info

| | |
|---|---|
| .NET | .NET 8 (`net8.0`) |
| Platform | ASP.NET Core |
| UI framework | **Razor Pages** + TypeScript (bundled with Rollup) |
| EasyQuery for .NET | **7.5.0** — `Korzh.EasyQuery.AspNetCore`, `Korzh.EasyQuery.EntityFrameworkCore.Relational`, `Korzh.EasyQuery.EntityFrameworkCore.Identity`, `Korzh.EasyQuery.DataExport`, `Korzh.EasyQuery.RazorUi`, `Korzh.EasyQuery.SqLiteGate`, `Korzh.EasyQuery.SqlServerGate` |
| EasyData exporters | 1.6.0 — `EasyData.Exporters.PdfSharp` (PDF), `EasyData.Exporters.ClosedXML` (Excel) |
| EasyQuery.JS | **7.5.1** from npm (`@easyquery/*` 7.5.1, `@easydata/*` 1.6.0); charts by Google Charts |
| TypeScript tooling | TypeScript 5.9, Rollup 4 (`rollup.config.js`) |
| Query engine | SQL (`UseSqlManager()`) |
| Data model source | EF Core `DbContext` (`UseDbContextWithoutIdentity<AppDbContext>`) |
| EasyQuery API | Middleware — `app.MapEasyQuery(...)`, endpoint `/api/adhoc-reporting` |
| Authentication | ASP.NET Core Identity |
| Database | SQLite through EF Core 8 (SQL Server is prepared in commented code) |

## Project structure

| File / folder | What's inside |
|---|---|
| `ts/adhoc-reporting.ts` | **The page script**: `ReportView` options and initialization |
| `ts/styles.js` | Entry point for the CSS bundle (imports EasyQuery/EasyData styles from `node_modules`) |
| `package.json`, `rollup.config.js`, `tsconfig.json` | The front-end build. Output: `wwwroot/js/adhoc-reporting(.min).js`, `wwwroot/css/adhoc-reporting(.min).css` |
| `build-easyreport.bat`, `watch.bat` | Shortcuts for `npm run build` / `npm run watch` |
| `.npmrc` | Commented settings for the Korzh MyGet feed (pre-release packages) |
| `Program.cs` | Services (DbContext, Identity, EasyQuery + exporters), the `MapEasyQuery` middleware, demo DB initialization |
| `Pages/Index.cshtml` | The page markup; includes `~/js/adhoc-reporting.js`. `[Authorize]` in `Index.cshtml.cs` |
| `Services/ReportStore.cs`, `Services/DefaultReportGenerator.cs` | Per-user report storage and the seed reports generator |
| `Models/`, `Data/`, `Areas/Identity/`, `App_Data/` | Same as in Razor.AdHocReporting |

## How to start

1. Install the [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) (or later) and [Node.js](https://nodejs.org/) (LTS; `npm` must be in `PATH`).
2. Put your EasyQuery license key(s) into `appsettings.json`:

   ```json
   "EasyQuery": {
     "LicenseKey": "<your .NET key>",
     "JSLicenseKey": "<your EasyQuery.JS key, if it's a separate one>"
   }
   ```

   or into user secrets (run in the project folder):

   ```bash
   dotnet user-secrets set "EasyQuery:LicenseKey" "<your key>"
   ```

   > **NB:** this project uses the same `UserSecretsId` as Razor.AdHocReporting, so the secrets you set for one of them apply to both.

   You can get a trial key in the [Korzh.com Client's Area](https://account.korzh.com) after registering an account.

3. Run the project:

   ```bash
   dotnet run
   ```

   The `PreBuild` target in the `.csproj` runs `npm install` and `npm run build` before every build, so the first build takes a while. You can also open `../Samples.AspNetCore.sln` and start `EqDemo.AspNetCoreRazorTypeScript.AdhocReporting`.
4. Open https://localhost:5001 (or http://localhost:5000) and log in with the default user:

   * login: **demo@korzh.com**
   * password: **demo**

   or register a new account. The demo user can view the reports but not manage them — register another account to create, save and remove reports.

On the first run the application creates the SQLite database `eqdemo-sqlite.db`, seeds it with the demo data, and creates the default user with a set of reports. To reset everything, stop the app and delete the `eqdemo-sqlite.db*` files. To reset only the demo user and its reports, set `"resetDefaultUser": true` in `appsettings.json` for one start.

### Working on the TypeScript code

While editing `ts/adhoc-reporting.ts`, keep Rollup running in watch mode in a separate terminal:

```bash
npm run watch
```

and just reload the page after each change. If you don't want `npm` to run on every `dotnet build`, remove the `PreBuild` target from the `.csproj` and build the script manually with `npm run build`.

## Next steps

### Connect the sample to your database

The server side is the same as in Razor.AdHocReporting, so follow
[its steps](../Razor.AdHocReporting/README.md#connect-the-sample-to-your-database): a second
`DbContext` for your data, `options.UseDbContext<...>()` in `MapEasyQuery`, Northwind and the demo
seeding out of `AppDbContext`, the seed reports out of `App_Data/Seed`. Two names differ here:

* the users-and-reports database is `ConnectionStrings:EqDemoSqLite` (not `EqDemoDb`) — in
  `appsettings.json`, `Program.cs` and `Data/DbInitializeExtensions.cs`;
* the scaffolding command needs `Microsoft.EntityFrameworkCore.Tools`, which this project already
  references, and a first build — which here also runs `npm install` and `npm run build`.

### Add EasyQuery to your own project

The server part — packages, `Program.cs`, the report storage — is the same as in
[Razor.AdHocReporting](../Razor.AdHocReporting/README.md#add-easyquery-to-your-own-project); do
steps 1–4 there. Leave out `UseSessionCache()` and the `StoreModelInCache` / `StoreQueryInCache`
options if you don't use sessions: this sample works without them. The difference is the UI,
which comes from npm instead of the CDN:

1. **Install the npm packages** in the folder of your project:

   ```bash
   npm install @easyquery/core @easyquery/ui @easyquery/enterprise @easydata/core @easydata/ui
   ```

   Use the versions from this sample's [`package.json`](package.json): `@easyquery/*` in line with
   the `Korzh.EasyQuery.*` NuGet packages, `@easydata/*` with the `EasyData.*` ones.
2. **Add the page script.** Copy [`ts/adhoc-reporting.ts`](ts/adhoc-reporting.ts) and
   [`ts/styles.js`](ts/styles.js). In the script, remove the `.setAppName(...)` and
   `.useBackend(...)` lines — they tell the library that it runs in this sample — and keep
   `.useEndpoint(...)` in line with `options.Endpoint` on the server.
3. **Bundle it.** If your front end has no build yet, copy [`rollup.config.js`](rollup.config.js),
   [`tsconfig.json`](tsconfig.json) and the `devDependencies` and `scripts` of `package.json`. The
   output is `wwwroot/js/adhoc-reporting.js` and `wwwroot/css/adhoc-reporting.css`. To build it with
   every `dotnet build`, copy the `PreBuild` target from the `.csproj` too. With a bundler of your
   own (Vite, webpack, esbuild…), just point it at `ts/adhoc-reporting.ts`.
4. **Add the page.** Copy [`Pages/Index.cshtml`](Pages/Index.cshtml) as e.g. `Pages/Reports.cshtml`,
   with an `[Authorize]` page model. It includes the two bundles and Google Charts. The same notes
   as for the Razor sample apply to it: the `Styles` section, Bootstrap 4 attributes and the
   `isDefaultUser` check, which here disables the ⚙ menu for the sample's demo user.
5. **Set the license key**, as described in [License key](../README.md#license-key).

More in the docs: [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project).

### Change the sample

#### Change the report view options

All client-side settings are in `ts/adhoc-reporting.ts` (typed as `ReportViewOptions`):

```ts
const viewOptions: ReportViewOptions = {
    calcTotals: true,                                   // totals row in the result
    enableExport: true,
    serverExporters: ['pdf', 'excel', 'excel-html', 'csv'],
    syncReportOnChange: true,                           // save on each change
    widgets: {
        queryPanel: {
            minDate: '2025-01-01',                      // limits of the date pickers in conditions
            maxDate: 'today'
        }
    },
    result: {
        showChart: true,
        paging: { pageSize: 30 }
    },
    loadModelOnStart: true,
    defaultModelId: 'adhoc-reporting'
};
```

Rebuild (`npm run build`, or just `dotnet build`) after changing it.

#### Update EasyQuery.JS or use pre-release packages

* Change the `@easyquery/*` and `@easydata/*` versions in `package.json` and run `npm install`. Keep them in line with the `Korzh.EasyQuery.*` NuGet versions in the `.csproj`.
* To get pre-release builds from the Korzh MyGet feed, uncomment the two registry lines in `.npmrc`.

#### Who can manage reports

This sample decides it in `Pages/Index.cshtml`: the ⚙ menu is disabled for the default user (`isDefaultUser`) and enabled for everyone else. The server side is protected separately by `options.UseDefaultAuthProvider(...)` in `Program.cs`: by default *New / Save / Remove* require the `eq-manager` role, which every newly registered user gets. To change the server rule, uncomment one of the lines there:

```csharp
options.UseDefaultAuthProvider((provider) => {
    provider.RequireAuthorization(EqAction.NewQuery, EqAction.SaveQuery, EqAction.RemoveQuery);
    //provider.RequireRole(DefaultEqAuthProvider.EqManagerRole, EqAction.NewQuery, EqAction.SaveQuery, EqAction.RemoveQuery);
});
```

#### Everything else — same as Razor.AdHocReporting

The server part is identical, so these recipes from the [Razor.AdHocReporting README](../Razor.AdHocReporting/README.md#next-steps) apply here as well:

* switching to SQL Server (uncomment in `Program.cs`, `Data/DbInitializeExtensions.cs`, `appsettings.json`). Note that here the SQLite connection string is named `EqDemoSqLite` and the SQL Server one `EqDemoDb`;
* showing each user only their own data (the commented lines in `AddPreFetchTunerWithHttpContext`);
* loading the model directly from the DB schema (the commented `.RegisterDbGate<...>()` lines) or from a saved file;
* tuning the DbContext-based model, the seed reports, and the default user.

Two server-side differences to be aware of: this project does not register the session cache (`UseSessionCache()`, `StoreModelInCache`, `StoreQueryInCache`) and does not set up CORS.

## Links

* [Live demo of ad-hoc reporting](https://demo.korzh.com/easyquery-asp-net-core-razor/adhoc-reporting?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* [All samples for ASP.NET Core MVC/Razor](../README.md)
* [All EasyQuery samples](https://github.com/easyquery)
* [EasyQuery home page](https://korzh.com/easyquery?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc)
* Want a full-scale solution for data management and ad-hoc reporting? Try [EasyReport Starter Kit](https://korzh.com/easy-report-starter-kit?utm_source=github&utm_medium=sample&utm_campaign=eq-samples-aspnetcore-razor-mvc).
