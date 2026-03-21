# NuGet (nuget)
NuGet is the package manager for .NET, providing a centralized repository for developers to discover, share, and consume reusable code libraries. The NuGet developer platform exposes a set of HTTP APIs that enable programmatic access to package search, metadata retrieval, content download, catalog browsing, and package publishing against the nuget.org feed.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/nuget/refs/heads/main/apis.yml)

## Scope

- **Type:** Contract
- **Position:** Consuming
- **Access:** 3rd-Party

## Tags:

 - Package Management, .NET, Packages, Dependencies, Software Distribution

## Timestamps

- **Created:** 2025-03-05
- **Modified:** 2026-03-20

## APIs

### NuGet Server API
The NuGet Server API is a set of HTTP endpoints used to download packages, fetch metadata, publish new packages, and perform other operations against a NuGet package source. The V3 API uses JSON as its underlying media type and is accessed through a service index located at https://api.nuget.org/v3/index.json, which enumerates all available resources and capabilities. Key resources include SearchQueryService for searching packages, RegistrationsBaseUrl for fetching package metadata, PackageBaseAddress for downloading package content, and PackagePublish for pushing new packages to the feed.

**Human URL:** [https://learn.microsoft.com/en-us/nuget/api/overview](https://learn.microsoft.com/en-us/nuget/api/overview)


#### Tags:

 - Package Management, .NET, Packages, Registry, Dependencies, Software Distribution

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/overview)
- [Documentation](https://learn.microsoft.com/en-us/nuget/api/service-index)
- [GitHub Repository](https://github.com/NuGet/NuGetGallery)
- [OpenAPI](openapi/nuget-server-api-openapi.yml)

### NuGet Catalog API
The NuGet Catalog API is an append-only resource that records the full history of all package events on nuget.org, including packages being added, modified, listed, unlisted, and deleted. It provides a chronologically ordered log of every change to the package source, enabling consumers to build and maintain their own local copy of the entire set of packages available on nuget.org. The Catalog API is particularly useful for building search indexes, maintaining package mirrors, and auditing the history of package operations.

**Human URL:** [https://learn.microsoft.com/en-us/nuget/api/catalog-resource](https://learn.microsoft.com/en-us/nuget/api/catalog-resource)


#### Tags:

 - Package Management, .NET, Catalog, Event Log, Packages

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/catalog-resource)
- [OpenAPI](openapi/nuget-catalog-api-openapi.yml)

### NuGet Search API
The NuGet Search API allows clients to query for packages available on a NuGet package source using the SearchQueryService resource found in the service index. It supports filtering by keyword, target framework, prerelease status, and package type, and returns paginated results with package metadata including versions, descriptions, download counts, and dependency information. This API powers the search experience in the NuGet client tools within Visual Studio, the dotnet CLI, and the nuget.org website, enabling developers to discover and evaluate packages for their .NET projects.

**Human URL:** [https://learn.microsoft.com/en-us/nuget/api/search-query-service-resource](https://learn.microsoft.com/en-us/nuget/api/search-query-service-resource)


#### Tags:

 - Package Management, .NET, Search, Discovery, Packages

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/search-query-service-resource)
- [OpenAPI](openapi/nuget-search-api-openapi.yml)

### NuGet Package Metadata API
The NuGet Package Metadata API, accessed through the RegistrationsBaseUrl resource, provides detailed metadata about packages available on a NuGet package source. It returns registration information including all available versions of a package, their dependencies, download URLs, descriptions, authors, license information, and deprecation status. The registration data is organized into pages and leaves, allowing efficient retrieval of metadata for packages that may have hundreds of published versions. This API is used by NuGet clients to resolve dependencies and display package details.

**Human URL:** [https://learn.microsoft.com/en-us/nuget/api/registration-base-url-resource](https://learn.microsoft.com/en-us/nuget/api/registration-base-url-resource)


#### Tags:

 - Package Management, .NET, Metadata, Package Registry, Versions

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/registration-base-url-resource)
- [OpenAPI](openapi/nuget-package-metadata-api-openapi.yml)

### NuGet Package Content API
The NuGet Package Content API, accessed through the PackageBaseAddress resource, allows clients to download package content files (.nupkg) and package manifests (.nuspec) from a NuGet feed. It uses a flat container structure where packages are organized by lowercase package ID and version, providing direct access to the binary package files. This API also exposes a version index endpoint that lists all available versions for a given package ID, enabling clients to enumerate and download specific package versions efficiently.

**Human URL:** [https://learn.microsoft.com/en-us/nuget/api/package-base-address-resource](https://learn.microsoft.com/en-us/nuget/api/package-base-address-resource)


#### Tags:

 - Package Management, .NET, Package Download, Content, NuPkg

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/package-base-address-resource)
- [OpenAPI](openapi/nuget-package-content-api-openapi.yml)

## Common Properties

- [Portal](https://learn.microsoft.com/en-us/nuget/)
- [Documentation](https://learn.microsoft.com/en-us/nuget/api/overview)
- [Website](https://www.nuget.org/)
- [PrivacyPolicy](https://privacy.microsoft.com/en-us/privacystatement)
- [TermsOfService](https://www.nuget.org/policies/Terms)
- [Blog](https://devblogs.microsoft.com/nuget/)
- [Login](https://www.nuget.org/users/account/LogOn)

## Maintainers

**FN:** API Evangelist

**Email:** info@apievangelist.com
