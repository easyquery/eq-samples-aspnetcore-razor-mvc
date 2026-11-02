# Mvc.DataFiltering.Auth

An ASP.NET Core MVC sample that implements the **data-filtering scenario for a multi-user application**: users log in, filter the Orders list with the EasyQuery filter bar, and their saved filters are kept **in the database, separately for each user**.

It extends the basic [Mvc.DataFiltering](../Mvc.DataFiltering/README.md) sample with:

* **ASP.NET Core Identity** with Register / Login / Logout pages. Users and roles live in the same database as the demo data (`AppDbContext` inherits `IdentityDbContext`). The Orders page is available to logged-in users only (`[Authorize]` on `OrderController`).
* **`DbQueryStore`** — a custom `IQueryStore` implementation that saves queries to the `UserQueries` table and binds each of them to the current user's ID. Different users can store a query under the same ID (e.g. `LastQuery`) — the table has a composite primary key (`Id` + `OwnerId`).
* **Automatic query loading** — the page is configured with `loadQueryOnStart: true` and `defaultQueryId: 'LastQuery'`, so the last query saved by the current user is loaded when the page opens.

Everything else is the same as in the basic sample: the filter bar, the full-text search box, and the result list rendered on the server by a Razor partial view. The EasyQuery API endpoints are ordinary MVC actions in `OrderController` (no `MapEasyQuery` middleware).

## Tech info

| | |
|---|---|
| .NET | .NET 8 (`net8.0`) |
| Platform | ASP.NET Core |
| UI framework | **MVC** (controllers + Razor views); Razor Pages only for the Identity UI |
| EasyQuery for .NET | **7.5.0** — `Korzh.EasyQuery.AspNetCore`, `Korzh.EasyQuery.Linq` |
| EasyQuery.JS | **7.5.1**, loaded from CDN (`eq.enterprise.min.js`, `eq.core.min.css`) |
| Query engine | LINQ — filters are applied to an `IQueryable<Order>` via `DynamicQuery()` |
| Data model source | The `Order` class and its navigation properties (`UseEntity`) |
| Authentication | ASP.NET Core Identity (EF Core stores, default UI) |
| Database | SQLite through EF Core 8 (SQL Server is prepared in commented code) |
| Query storage | `DbQueryStore` — the `UserQueries` table, per user |

## Project structure

| File / folder | What's inside |
|---|---|
| `Program.cs` | Service registration (DbContext, Identity, `AddEasyQuery()`), the request pipeline, demo DB initialization |
| `Controllers/OrderController.cs` | EasyQuery endpoints under `/data-filtering` (model, value lists, query load/save, fetch). Marked with `[Authorize]` |
| `Services/DbQueryStore.cs` | The query store that keeps each user's queries in the database |
| `Models/UserQuery.cs` | The entity for a stored query (with `OwnerId` pointing to the Identity user) |
| `Models/NWind/` | Entity classes of the demo (Northwind) database |
| `Data/AppDbContext.cs` | The DbContext; inherits `IdentityDbContext`, so it contains the Identity tables too |
| `Data/DbInitializeExtensions.cs` | On the first run: creates the DB, seeds demo data, adds the `eq-manager` role and the default user |
| `Areas/Identity/` | Login, Register and Logout pages |
| `Views/Order/Orders.cshtml` | The page: FilterBar, text search, Load/Save Query buttons, EasyQuery.JS initialization |
| `Views/Order/_OrderListPartial.cshtml` | The server-rendered result list |
| `Views/Shared/_LoginPartial.cshtml` | The login/logout part of the navbar |
| `App_Data/EqDemoData.zip` | Demo data used to seed the database |

## How to start

1. Install the [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) (or later).
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

   > **NB:** without a valid EasyQuery.JS license key, EasyQuery.JS falls back to storing queries in the browser's local storage, so `DbQueryStore` is **not** called on Save / Load Query. If your EasyQuery.JS key is separate from the .NET one, add it as `EasyQuery:JSLicenseKey`.

3. Run the project:

   ```bash
   dotnet run
   ```

   or open `../Samples.AspNetCore.sln` and start `EqDemo.AspNetCoreMvc.DataFiltering.Auth`.
4. Open https://localhost:5003 (or http://localhost:5002).

On the first run the application creates the SQLite database `eqdemo-auth-sqlite.db`, seeds it with the demo data and adds the default user:

* login: **demo@korzh.com**
* password: **demo**

You can also register your own account — every user gets their own set of saved queries.

Try it: log in, add some conditions in the filter bar, press **Apply** and then **Save Query**. Reload the page (or log in again later) — the saved query is loaded automatically. Log in as another user and you'll see that user's own `LastQuery` instead.

To reset the database, stop the application and delete the `eqdemo-auth-sqlite.db*` files.

## Next steps

### Connect the sample to your database

`AppDbContext` holds the Northwind tables, the users and roles (it's an `IdentityDbContext`) and
the saved queries (`UserQueries`). Leave users and queries in the sample's SQLite file and give
your data a `DbContext` of its own:

1. **Connection string.** Add one for your database to `ConnectionStrings` in `appsettings.json`,
   e.g. `"MyDb": "..."`. `EqDemoDb` stays as the database of users and queries.
2. **Your `DbContext`.** Generate it from your database (build the project once first; the example
   is for SQL Server) and register it in `Program.cs`:

   ```bash
   dotnet tool install --global dotnet-ef
   dotnet ef dbcontext scaffold "<your connection string>" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models/MyDb --namespace EqDemo.Models.MyDb --context MyDataContext --context-dir Data --context-namespace EqDemo --no-onconfiguring
   ```

   ```csharp
   builder.Services.AddDbContext<MyDataContext>(options =>
       options.UseSqlServer(configuration.GetConnectionString("MyDb")));
   ```

3. **The controller.** In `Controllers/OrderController.cs`, take `MyDataContext` instead of
   `AppDbContext` in the constructor and change the entity — `EasyQueryManagerLinq<Order>`, the two
   `Orders.Include(...)` queries, the `FullTextSearchOptions` filter — exactly as in
   [Mvc.DataFiltering](../Mvc.DataFiltering/README.md#connect-the-sample-to-your-database), step 4.
   `DbQueryStore` keeps using `AppDbContext`. Then change the list in `Views/Order/_OrderListPartial.cshtml`.
4. **Remove Northwind from `AppDbContext`.** Delete the `NWind` region and the `OrderDetail` lines
   of `OnModelCreating` in `Data/AppDbContext.cs`, then the `Models/NWind/` folder. Keep
   `UserQueries` and its key.
5. **Demo seeding.** In `Data/DbInitializeExtensions.cs`, replace everything from the seeding-marker
   comment down to the end of the `if (context.Database.EnsureCreated()) { ... }` block with a single
   `context.Database.EnsureCreated();`, so the role and the default user are still created. Delete
   the old `eqdemo-auth-sqlite.db*` files, if any.

### Add EasyQuery to your own project

This is the [Mvc.DataFiltering](../Mvc.DataFiltering/README.md#add-easyquery-to-your-own-project)
filter bar for an application that already has users (ASP.NET Core Identity): follow the steps
there, and add the per-user query storage:

1. **Protect the controller** with `[Authorize]`, as `OrderController` here is.
2. **Store queries per user.** Copy [`Services/DbQueryStore.cs`](Services/DbQueryStore.cs) and
   [`Models/UserQuery.cs`](Models/UserQuery.cs), change their namespaces, and in `DbQueryStore`
   replace `AppDbContext` with the `DbContext` of your users. Add the queries to that context,
   with the composite key that lets every user have their own `LastQuery`:

   ```csharp
   public DbSet<UserQuery> UserQueries { get; set; }

   protected override void OnModelCreating(ModelBuilder modelBuilder)
   {
       base.OnModelCreating(modelBuilder);
       modelBuilder.Entity<UserQuery>().HasKey(q => new { q.Id, q.OwnerId });
   }
   ```

   Create the table (`ApplicationDbContext` is the name the project templates give the Identity
   context):

   ```bash
   dotnet ef migrations add AddUserQueries --context ApplicationDbContext
   dotnet ef database update --context ApplicationDbContext
   ```

   In the controller's constructor, use the new store: `options.UseQueryStore(_ => new DbQueryStore(services));`.
3. **Load the user's last query on start**, if you want to: `loadQueryOnStart: true` and
   `defaultQueryId: 'LastQuery'` in the `viewOptions` of `Views/Order/Orders.cshtml`.

What not to copy: the Identity setup itself — `AddDefaultIdentity(...)` with its password rules,
`Areas/Identity/`, `_LoginPartial.cshtml`, and the default user and the `eq-manager` role in
`Data/DbInitializeExtensions.cs`. That's this sample's own login, not part of EasyQuery; your
application already has one.

More in the docs: [From a sample to your own project](https://korzh.com/easyquery/docs/getting-started/from-sample-to-your-own-project).

### Change the sample

#### Move the demo database to SQL Server (uncomment)

1. `appsettings.json` — comment the SQLite `EqDemoDb` connection string and uncomment the `(localdb)\\MSSQLLocalDB` one (or put your own).
2. `Program.cs` — replace `options.UseSqlite(...)` with the commented `options.UseSqlServer(...)` line.
3. `Data/DbInitializeExtensions.cs` — do the same inside `DbInitializer.Create(...)`.

#### Don't load the query automatically / use another default query

In `Views/Order/Orders.cshtml`:

```js
var viewOptions = {
    loadModelOnStart: true,
    loadQueryOnStart: false,        // set to false to always start with an empty filter
    defaultQueryId: 'LastQuery',    // or the ID of any other saved query
    ...
};
```

The **Save Query** / **Load Query** handlers at the end of the same file also use the `LastQuery` ID — change them together.

#### Restrict the data each user can see

`DbQueryStore` separates the *queries* by user, but all users filter the same Orders. To limit the *data*, add a regular LINQ condition in `ApplyQueryFilter` (`OrderController.cs`) — it's combined with the user's filter and never shown in the UI:

```csharp
var userName = User.Identity?.Name;

var orders = _dbContext.Orders
          .Include(o => o.Customer)
          .Include(o => o.Employee)
          .Where(o => o.Employee.Email == userName)   // an example: map users to employees your own way
          .AsQueryable();
```

(The demo Northwind `Employee` has no link to Identity users — the condition above is a pattern to adapt, not something that works out of the box.)

#### Store queries differently

`options.UseQueryStore(...)` in the `OrderController` constructor selects the storage:

* Back to files (shared by all users): `options.UseQueryStore(_ => new FileQueryStore("App_Data"));`
* Your own storage — implement `IQueryStore` the way `DbQueryStore` does. Its `ApplyUserGuard()` method is the one place where queries are scoped to a user; change it if queries should be shared by a team, a tenant, etc.

#### Change the account rules

* Password rules and cookie lifetime — the `AddDefaultIdentity<IdentityUser>(...)` and `ConfigureApplicationCookie(...)` calls in `Program.cs`.
* The default user — the `_defaultUserEmail` / `_defaultUserPassword` constants in `Data/DbInitializeExtensions.cs`.
* New users are added to the `eq-manager` role on registration (`Areas/Identity/Pages/Account/Register.cshtml.cs`). This sample doesn't restrict anything by that role, but the ad-hoc reporting samples do — see [Razor.AdHocReporting](../Razor.AdHocReporting/README.md).

#### Everything from the basic sample

Filtering another entity, tuning the model with attributes, changing the full-text search fields, paging and the result markup work exactly as described in the [Mvc.DataFiltering README](../Mvc.DataFiltering/README.md#next-steps).
