# Modernization Plan: PhotoAlbum Azure Modernization

**Project**: PhotoAlbum

---

## Technical Framework

- **Language**: C#
- **Framework**: ASP.NET Core 9.0
- **Build Tool**: .NET SDK / MSBuild
- **Database**: SQL Server LocalDB
- **Key Dependencies**: Entity Framework Core 9, SixLabors.ImageSharp

---

## Overview

> This modernization prepares PhotoAlbum to run in Azure while retaining its
> photo gallery behavior. The application currently stores photo files locally,
> connects to SQL Server LocalDB, and keeps application settings in local JSON
> configuration. The modernized application will:
>
> - Store uploaded photos in Azure Blob Storage.
> - Use Azure SQL Database for photo metadata.
> - Externalize non-secret settings to Azure App Configuration.
> - Run on the supported .NET 10 LTS runtime.
>
> The work is planned in phases: upgrade the runtime, migrate the application’s
> storage and configuration integrations, then verify the migrated behavior and
> review dependencies for known vulnerabilities. Existing infrastructure
> definitions are reused; provisioning and deployment are not part of this plan.

---

## Migration Impact Summary

| App | Original | Azure target | Authentication | Notes |
|-----|----------|--------------|----------------|-------|
| PhotoAlbum | Disk files | Blob Storage | Managed identity | Photos |
| PhotoAlbum | LocalDB | Azure SQL | Managed identity | Metadata |
| PhotoAlbum | Local JSON | App Configuration | Managed identity | Settings |

---

## Assessment Findings Reviewed

- The local/network I/O finding is present in the photo upload, retrieval, and
  deletion paths and is addressed by the Blob Storage migration.
- The SQL, connection string, Windows authentication, and legacy connection
  configuration findings are supported by the LocalDB connection settings and
  are addressed by the Azure SQL migration.
- The local application configuration finding is present in `appsettings.json`
  and is addressed by externalizing non-secret settings.
- The repository infrastructure does not currently define an Azure App
  Configuration resource. An existing resource must be supplied separately;
  provisioning it is outside this plan.
- The hardcoded-sensitive-data finding was not confirmed: the current login
  page reads the password from configuration and does not contain a hardcoded
  password. It is not treated as a migration issue.
- Static assets are present in the web project but remain application assets;
  no static-content migration was requested.

---

## Open Questions & Questionnaire

- [x] Q: Should the plan include environment or infrastructure provisioning? → A: No; reuse the repository-defined infrastructure in `infra/main.bicep`.
- [x] Q: Should the plan include integration testing? → A: Yes; use mocked dependencies because no provisioned test environment or resource identifiers were provided.
- [x] Q: Should the plan include security and CVE remediation? → A: Yes; use the default dependency scan and remediation scope.
- [x] Q: Which Azure deployment target should the plan use? → A: No deployment; deployment was not requested.
- [x] Q: Should the plan include containerization? → A: No separate containerization task; deployment was not requested and a Dockerfile already exists.
