# MORYX API Standards

## 1. Controller conventions

### 1.1 Required attributes

**Every MORYX controller must declare `[ApiController]`, `[Route("api/moryx/{resource}/")]`, and `[Produces("application/json")]`**

`[ApiController]` activates automatic model validation. Without it, if a request body fails validation, ASP.NET Core calls your action method anyway with `ModelState.IsValid == false`. In that case, the developer has to check that value. If that value is not checked, the endpoint accepts corrupt data and returns `200 OK`. With `[ApiController]`, the framework rejects the request before your code even runs, returning a structured `400 Bad Request`.

`[Route]` pins the URL directly in code. ASP.NET Core's default convention derives URLs from the controller class name, so renaming `ResourceManagementController` to `ResourceController` would silently break the URL in production.

`[Produces("application/json")]` tells Swashbuckle to declare `application/json` as the only content type in the OAS document. Without it, the document may declare `*/*`, which can cause the Angular code generator to emit branching logic for content types you never intended to support.

**More information:**

- [Microsoft Docs — ApiController attribute](https://learn.microsoft.com/en-us/aspnet/core/web-api/#apicontroller-attribute)
- [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457.html)

---

### 1.2 Route format

**All MORYX routes must follow `api/moryx/{resource}/`**
**No version segment unless a breaking change makes it unavoidable.**

The `/api/` prefix separates API traffic from UI routes on the same host — without it, a future Angular route could silently shadow an API endpoint. The `/moryx/` segment namespaces MORYX within the host application, so there's no collision with other subsystems that might use the same resource names.

There's no version segment by default. Adding `/v1/` from the start creates a commitment to keep those routes alive forever, or forces a coordinated cutover when v2 arrives. The proper approach is to treat the current route as the current contract and only add a version when a breaking change makes it unavoidable.

**More information:**

- [Microsoft REST API Guidelines — Versioning](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md#versioning)

---

### 1.3 `ControllerBase`, not `Controller`

**All MORYX controllers must extend `ControllerBase`**

`Controller` extends `ControllerBase` with everything needed for Razor views: `ViewData`, `TempData`, `View()`, `PartialView()`, and the `IDisposable` implementation that Razor requires. None of that belongs in a JSON API. More practically, `Controller` pulls in `Microsoft.AspNetCore.Mvc.ViewFeatures`, which can cause startup failures in minimal-API hosts that don't include Razor support. Using `ControllerBase` keeps the surface area clean and the dependency graph honest.

**More information:**

- [Microsoft Docs — ControllerBase vs Controller](https://learn.microsoft.com/en-us/aspnet/core/web-api/#controllerbase)

---

### 1.4 Constructor injection and null checks

**Inject all services via the constructor.**
**Null-check every required service with `?? throw new ArgumentNullException(...)`.**

Constructor injection makes the controller's dependencies visible as a compile-time contract — you can read the constructor and know exactly what the controller needs. If a required service isn't registered, the dependency injection container fails at startup with a clear message, rather than throwing a `NullReferenceException` somewhere deep in a request.

The null check covers the case where a service is registered but a factory method returns `null` due to a bug. Without it, the failure happens at the first use site, with a stack trace that points nowhere useful.

**More information:**

- [Microsoft Docs — Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)

---

## 2. HTTP method semantics

**Use `GET`, `POST`, `PUT`, and `DELETE` strictly according to their RFC 9110 definitions.**
**Safe methods must not mutate state.**
**Idempotent methods must be safe to retry.**
**`POST` must never be used for read-only queries.**

Safe methods (`GET`, `HEAD`) are defined as having no observable side effects. If a `GET` endpoint mutates data, a cache might serve a stale response while the mutation is silently dropped.

Idempotent methods (`GET`, `PUT`, `DELETE`) can be repeated without changing the outcome after the first call. This allows clients and load balancers to safely retry requests when a response is lost to a network failure. `POST` is not idempotent, so a retry might create a duplicate resource.

Don't use `POST` for queries just because the query object feels too large for a URL — `[FromQuery]` handles complex objects fine, and if you ever genuinely hit URL-length limits, the right answer is a saved-search resource, not overloading `POST` with read semantics.

**More information:**

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [MDN Web Docs — HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

---

## 3. Route design

### 3.1 Resources, not operations

**Route segments must identify resources.**

**Putting a HTTP method in the URL path is forbidden.**

Routes identify things, not actions. The HTTP method is the verb; the URL is the noun. `DELETE /api/moryx/resources/42` is unambiguous. `POST /api/moryx/deleteResource/42` contradicts itself.

Once `POST /deleteProduct/{id}` exists alongside `DELETE /products/{id}`, both tend to live forever because clients have linked to both.

**More information:**

- [Microsoft Learn — Web API Design Best Practices: Organize the API around resources](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#organize-the-api-around-resources)
- [REST API Tutorial — URI Naming Conventions and Best Practices](https://restfulapi.net/resource-naming/)

---

### 3.2 Collection queries

**Filtered collection queries should use `GET` with `[FromQuery]` binding.**
**For queries that are too copmlex to express cleanly in a query string, `POST` with the query in the request body is acceptable.**

`GET` is the default choice: it sticks to HTTP semantics, responses can be cached by browsers and in-process caches using the full URL as the cache key, and ASP.NET Core handles complex objects and arrays from query strings automatically.

When a query grows too large in a URL (e.g. deeply nested filters), you may switch to `POST` with the query object in the body.

**More information:**

- [RFC 9110 — POST](https://www.rfc-editor.org/rfc/rfc9110#section-9.3.3)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
- [Microsoft Docs — Model binding in ASP.NET Core — FromQuery](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/model-binding#sources)

---

## 4. HTTP status codes

**Every endpoint must declare every status code it can return via `[ProducesResponseType]`.**
**Each code must match its RFC 9110 definition.**

**201 Created** — `201` is for when a new resource was created. It also mandates a `Location` header pointing to the new resource, which gives clients a canonical URL without reconstructing it from the response body.

**204 No Content** — use this for successful `DELETE` responses. `200 OK` implies a response body representing the resource, which doesn't exist anymore.

**422 Unprocessable Content** — use this when the HTTP request was valid but the business logic failed. `400 Bad Request` is for syntactically malformed requests.

Declare every possible status code with `[ProducesResponseType]`. The Angular code generator reads the OAS document to generate a discriminated union of response types (one per declared status code). Undeclared codes fall through as untyped `HttpErrorResponse`, losing the structured error handling the standard is designed to provide.

**More information:**

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110#section-15)
- [RFC 4918 — HTTP Extensions for WebDAV](https://www.rfc-editor.org/rfc/rfc4918#section-11.2)
- [ng-openapi-gen GitHub](https://github.com/cyclosproject/ng-openapi-gen)
- [wikipedia - List of HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

---

## 4. Error responses

**All error responses must use `MoryxExceptionResponse` as the response body.**
**Raw strings as error bodies are forbidden.**

The reason is the code generation pipeline: if an error response body is a raw string, Swashbuckle emits `"type": "string"` in the OAS schema, and `ng-openapi-gen` generates `string` on the TypeScript side. The Angular client then has to parse the string manually, which defeats the point of having a typed API contract.

`MoryxExceptionResponse` includes a human-readable `title` and, in development builds, the exception type name in an `Exception` field. Return enough detail for legitimate clients, nothing more.

**Note:**

This is the current approach and is considered a temporary solution. In the future, error responses will be replaced with RFC compliant objects. RFC style problem details provide a standardized, extensible structure for API error responses.

**More information:**

- [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
- [OWASP — Improper Error Handling](https://owasp.org/www-community/Improper_Error_Handling)

---

## 5. Return types

### 5.1 `ActionResult<T>`

**Every endpoint that returns data must declare `ActionResult<T>` as its return type.**

Swashbuckle needs the generic type parameter to know what schema to emit for the `200 OK` response. With `IActionResult`, it can't infer anything and emits an empty schema, which `ng-openapi-gen` renders as `any` in TypeScript. That's not "less typed" — it actively breaks the generated client, making every consumer lose compile-time type safety.

`ActionResult<T>` lets you return either the value directly (`return model;`) or a typed error result (`return NotFound(new MoryxExceptionResponse{...})`), without any casts.

**More information:**

- [Microsoft Docs — ActionResult\<T\> type](https://learn.microsoft.com/en-us/aspnet/core/web-api/action-return-types#actionresultt-type)

---

### 5.2 Response model design

**Array properties must default to `[]`, never `null`.**

Array properties should default to `[]`, never `null`. In JSON, `null` and `[]` are different values even if they mean the same thing in C#. The OAS schema declares array fields as `"type": "array"`, and a `null` value violates that schema.

**More information:**

- [OpenAPI Specification 3.1 — Schema Object, nullable](https://spec.openapis.org/oas/v3.1.0#schema-object)

---

### 5.3 Server-Sent Events

SSE endpoints must handle module restarts and reconnect automatically.

Streaming endpoints must not assume that the underlying event source remains available for the entire lifetime of the HTTP connection. Modules can be restarted, temporarily become unavailable, or change their lifecycle state while clients remain connected.

To ensure reliable notification delivery, SSE endpoints should automatically reconnect to the event source when it becomes available again and continue streaming updates without requiring the client to establish a new connection.

This prevents stale UI state after temporary server interruptions and ensures that clients continue to receive notification updates after module restarts or lifecycle transitions.

**More information:**

- [Server-Sent Events Specification](https://html.spec.whatwg.org/multipage/server-sent-events.html)

---

## 6. Authorization

### 6.1 Permission constants

**All permission strings must be defined as constants in a `{Domain}Permissions` class, prefixed with `"Moryx.{Domain}."`.**
**The constant name must exactly match its string value.**

The constants class avoids copy-paste: if the string needs to change, there's one place to update. The prefix is a namespace — without it, two MORYX modules both defining `CanDelete` will share the same ASP.NET Core authorization policy.

**More information:**

- [Microsoft Docs — Policy-based authorization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies)

---

### 6.2 Per-endpoint `[Authorize]`

**`[Authorize(Policy = ...)]` must be applied to each endpoint individually**
**Controller-level authorization attributes are forbidden.**

A controller-level attribute applies the same permission to every endpoint, which inverts the principle of least privilege. If `CanDelete` is on the controller, a user with delete permission implicitly gains read and create access too.

**More information:**

- [Microsoft Docs — Role-based and policy-based authorization — Applying per-action](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/roles#applying-role-checks)
- [OWASP — Principle of Least Privilege](https://owasp.org/www-community/Access_Control)

---

## 7. OAS annotations

**Every endpoint must carry `[HttpMethod]`, a `[ProducesResponseType]` for each possible response code, `[Authorize]` (or `[AllowAnonymous]`), and an XML `<summary>` doc comment.**

Each annotation feeds a specific part of the OAS document, which becomes a specific part of the generated TypeScript. Missing `[ProducesResponseType]` means `any` return types or missing error branches. Missing `<summary>` means methods with no JSDoc comments in TypeScript, forcing developers to look up the controller source to understand what a method does.

**More information:**

- [OpenAPI Specification 3.0](https://spec.openapis.org/oas/v3.0.3)
