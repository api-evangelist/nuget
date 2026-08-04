# NuGet (nuget)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

NuGet is the package manager for .NET, providing a centralized repository for developers to discover, share, and consume reusable code libraries. The NuGet developer platform exposes a set of HTTP APIs that enable programmatic access to package search, metadata retrieval, content download, catalog browsing, and package publishing against the nuget.org feed.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/nuget/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/nuget/refs/heads/main/apis.yml)

## Tags

- Package Management
- .NET
- Packages
- Dependencies
- Software Distribution
- Registry

## Timestamps

- **Created:** 2025-03-05
- **Modified:** 2026-05-19

## APIs

### NuGet Server API

The NuGet Server API is a set of HTTP endpoints used to download packages, fetch metadata, publish new packages, and perform other operations against a NuGet package source. The V3 API uses JSON as its underlying media type and is accessed through a service index located at https://api.nuget.org/v3/index.json, which enumerates all available resources and capabilities.

- **Human URL:** [https://learn.microsoft.com/en-us/nuget/api/overview](https://learn.microsoft.com/en-us/nuget/api/overview)

#### Tags

- Package Management
- .NET
- Packages
- Registry
- Dependencies

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/overview)
- [Documentation](https://learn.microsoft.com/en-us/nuget/api/service-index)
- [GitHub Repository](https://github.com/NuGet/NuGetGallery)
- [OpenAPI](openapi/nuget-server-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/nuget-server-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nuget-server-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### NuGet Catalog API

The NuGet Catalog API is an append-only resource that records the full history of all package events on nuget.org, including packages being added, modified, listed, unlisted, and deleted. It provides a chronologically ordered log of every change to the package source, enabling consumers to build and maintain their own local copy of the entire set of packages available on nuget.org.

- **Human URL:** [https://learn.microsoft.com/en-us/nuget/api/catalog-resource](https://learn.microsoft.com/en-us/nuget/api/catalog-resource)

#### Tags

- Package Management
- .NET
- Catalog
- Event Log
- Packages

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/catalog-resource)
- [OpenAPI](openapi/nuget-catalog-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/nuget-catalog-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nuget-catalog-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### NuGet Search API

The NuGet Search API allows clients to query for packages available on a NuGet package source using the SearchQueryService resource found in the service index. It supports filtering by keyword, target framework, prerelease status, and package type, and returns paginated results with package metadata including versions, descriptions, download counts, and dependency information.

- **Human URL:** [https://learn.microsoft.com/en-us/nuget/api/search-query-service-resource](https://learn.microsoft.com/en-us/nuget/api/search-query-service-resource)

#### Tags

- Package Management
- .NET
- Search
- Discovery
- Packages

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/search-query-service-resource)
- [OpenAPI](openapi/nuget-search-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/nuget-search-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nuget-search-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### NuGet Package Metadata API

The NuGet Package Metadata API, accessed through the RegistrationsBaseUrl resource, provides detailed metadata about packages available on a NuGet package source. It returns registration information including all available versions of a package, their dependencies, download URLs, descriptions, authors, license information, and deprecation status.

- **Human URL:** [https://learn.microsoft.com/en-us/nuget/api/registration-base-url-resource](https://learn.microsoft.com/en-us/nuget/api/registration-base-url-resource)

#### Tags

- Package Management
- .NET
- Metadata
- Package Registry
- Versions

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/registration-base-url-resource)
- [OpenAPI](openapi/nuget-package-metadata-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/nuget-package-metadata-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nuget-package-metadata-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### NuGet Package Content API

The NuGet Package Content API, accessed through the PackageBaseAddress resource, allows clients to download package content files (.nupkg) and package manifests (.nuspec) from a NuGet feed. It uses a flat container structure where packages are organized by lowercase package ID and version, providing direct access to the binary package files.

- **Human URL:** [https://learn.microsoft.com/en-us/nuget/api/package-base-address-resource](https://learn.microsoft.com/en-us/nuget/api/package-base-address-resource)

#### Tags

- Package Management
- .NET
- Package Download
- Content
- NuPkg

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/nuget/api/package-base-address-resource)
- [OpenAPI](openapi/nuget-package-content-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/nuget-package-content-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/nuget-package-content-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [Portal](https://learn.microsoft.com/en-us/nuget/)
- [Documentation](https://learn.microsoft.com/en-us/nuget/api/overview)
- [Website](https://www.nuget.org/)
- [Privacy Policy](https://privacy.microsoft.com/en-us/privacystatement)
- [Terms of Service](https://www.nuget.org/policies/Terms)
- [Blog](https://devblogs.microsoft.com/nuget/)
- [Login](https://www.nuget.org/users/account/LogOn)
- [M C P Server](https://devblogs.microsoft.com/dotnet/nuget-mcp-server-preview/)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
