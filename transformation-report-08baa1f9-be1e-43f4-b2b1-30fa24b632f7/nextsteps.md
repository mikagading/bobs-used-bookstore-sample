# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build produced no errors across all projects after transformation. This indicates the migration to cross-platform .NET was completed successfully. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run the following commands from the solution root to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Ensure there are no warnings that could indicate compatibility issues, such as deprecated APIs or platform-specific calls that may have been silently retained.

---

## 2. Run Unit Tests

Execute the test project to confirm all existing tests pass on the new runtime:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the output for:
- Any failing tests
- Any skipped tests that may have been conditionally excluded during migration
- Runtime exceptions that differ from the original .NET Framework behavior

---

## 3. Verify Data Layer Behavior

Since `Bookstore.Data` handles data access, confirm the following:

- **Database connectivity**: Ensure connection strings in configuration files (e.g., `appsettings.json`) are correct and the target database is reachable.
- **Migrations**: If Entity Framework Core is in use, verify that existing migrations are compatible and apply cleanly:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

- **ORM behavior differences**: If the project was migrated from Entity Framework 6 to Entity Framework Core, test queries and relationships carefully, as there are known behavioral differences between the two.

---

## 4. Run and Smoke Test the Web Application

Start the web application locally and perform a basic functional walkthrough:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- Application starts without runtime errors
- All routes resolve correctly
- Authentication and authorization flows work as expected
- Static assets load properly
- Any middleware configured in `Program.cs` or `Startup.cs` behaves as intended

---

## 5. Review CDK Project

The `Bookstore.Cdk` project likely defines infrastructure. Confirm the following:

- All CDK dependencies are referencing the correct NuGet packages for the target .NET version
- Any environment-specific configuration values (e.g., region, account, resource names) are accurate
- Synthesize the CDK stack to validate the output:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

---

## 6. Check for Platform-Specific Code

Even without build errors, some APIs that existed in .NET Framework may have been replaced with cross-platform alternatives that behave differently at runtime. Search the codebase for the following:

- Usage of `System.Web` namespaces (should have been replaced)
- Windows Registry access (`Microsoft.Win32.Registry`)
- `HttpContext.Current` references
- Any P/Invoke calls or `DllImport` attributes targeting Windows-only native libraries

```bash
grep -rn "System.Web\|Registry\|HttpContext.Current\|DllImport" app/
```

---

## 7. Validate Configuration Files

Confirm that `appsettings.json` and any environment-specific variants (`appsettings.Development.json`, etc.) contain all settings that were previously held in `Web.config` or `App.config`. Pay particular attention to:

- Connection strings
- Application settings keys
- Custom configuration sections

---

## 8. Target Framework Confirmation

Verify each project is targeting the intended .NET version by inspecting each `.csproj` file:

```bash
grep -rn "TargetFramework" app/
```

All projects should consistently target the same framework version (e.g., `net8.0`) unless there is a specific reason for variation.