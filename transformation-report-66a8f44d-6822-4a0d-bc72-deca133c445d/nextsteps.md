# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas of the code that may behave differently under cross-platform .NET.

---

## 3. Run Unit Tests

Execute the test project to validate that existing business logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output carefully. Any failing tests should be investigated to determine whether the failure is due to a behavioral difference in cross-platform .NET versus the legacy framework.

---

## 4. Verify Data Layer Functionality

Since `Bookstore.Data` handles data access, verify the following:

- **Database provider compatibility**: Confirm that the database provider package (e.g., Entity Framework Core, Dapper, or another ORM) is compatible with the target .NET version.
- **Connection strings**: Ensure connection strings are correctly configured in the new project structure, typically in `appsettings.json` for .NET projects.
- **Migrations**: If Entity Framework Core is in use, verify that existing migrations are intact and apply cleanly:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

---

## 5. Validate the Web Application

Run the `Bookstore.Web` project locally and manually verify core functionality:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following areas:

- Application starts without runtime exceptions.
- All routes and pages load correctly.
- Authentication and authorization flows work as expected, if applicable.
- Static assets (CSS, JavaScript, images) are served correctly.
- Any middleware configured in `Program.cs` or `Startup.cs` behaves as intended.

---

## 6. Review Platform-Specific Code

Cross-platform .NET does not support certain Windows-specific APIs. Search the codebase for any usage of the following and replace or conditionally compile as needed:

- `System.Web` namespace references
- Windows Registry access (`Microsoft.Win32.Registry`)
- Windows-only file path assumptions (e.g., hardcoded backslashes)
- COM interop or P/Invoke calls targeting Windows-only libraries

---

## 7. Review the CDK Project

The `Bookstore.Cdk` project likely defines infrastructure. Verify the following:

- The AWS CDK or relevant SDK version referenced is compatible with the target .NET version.
- Any environment-specific configuration (regions, account IDs, resource names) is correctly set for the target deployment environment.
- Run a CDK synthesis to validate the infrastructure definitions without deploying:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

Or if using the CDK CLI:

```bash
cdk synth
```

---

## 8. Configuration and Environment Variables

Review `appsettings.json` and any environment-specific configuration files (`appsettings.Development.json`, `appsettings.Production.json`) to ensure:

- All required keys are present.
- Secrets are not stored in source-controlled configuration files; use environment variables or a secrets manager instead.
- Logging configuration is appropriate for each environment.

---

## 9. Final Smoke Test

Before deploying to a production environment, run the application against a staging environment that mirrors production as closely as possible. Validate end-to-end functionality including database connectivity, external service integrations, and expected application behavior.