# Security Integration Guidelines

These guidelines describe the security-relevant steps an integrator must take when building and deploying a product on top of the MORYX Framework.

The framework provides the building blocks for a secure application, but **the security of the final product is the responsibility of the integrator**. 

For a general overview of secure application development you may also refer to the [secure coding guidelines](https://learn.microsoft.com/en-us/dotnet/standard/security/secure-coding-guidelines) from Microsoft.

To report a vulnerability in the MORYX Framework itself, follow the process in [SECURITY.md](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/dev/SECURITY.md). Do **not** open a public GitHub issue for security reports.

---

## 1. Authentication and Authorization

MORYX ships a development-only `ExamplePolicyProvider` that exposes every endpoint anonymously. This configuration must never be used in a production deployment.

> **Do not deploy with `ExamplePolicyProvider` in production.** This configuration exposes every endpoint anonymously — anyone with network access can use every endpoint of your application. Under CRA Annex I, Part I §2(d), the product you ship must be **secure by default**: connecting to an IAM server is a mandatory requirement before any production deployment, not an optional hardening step.

For production, connect your application to a MORYX Access Management (IAM) server. It is built on top of [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity), the standard .NET solution for managing identities and access to resources, and integrates seamlessly with any MORYX application and its user interfaces:

- [Identity and Access Management overview](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/dev/docs/articles/moryx-accessmanagement/index.md) — the `Moryx.Identity` and `Moryx.Identity.AccessManagement` packages.
- [How to configure your application to connect to a running IAM server](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/dev/docs/articles/moryx-accessmanagement/how-to-integrate-in-your-application.md).
- [How to secure your endpoints and controllers with permissions](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/dev/docs/articles/moryx-accessmanagement/how-to-integrate-in-endpoints.md).

Key practices:

- Replace `ExamplePolicyProvider` with `MoryxAuthorizationPolicyProvider` and register the authentication/authorization middleware as described in the IAM integration guide.
- Protect every controller action with an `[Authorize(Policy = …)]` attribute and a dedicated permission — do not leave endpoints anonymous.
- Host the application and the IAM server under the same second-level domain so the shared authentication cookie works.

## 2. Credentials and Secrets

- The MORYX Framework ships **no default passwords**. All credentials belong to the systems you connect (IAM server, database, message brokers). Never introduce hard-coded credentials.
- Store connection strings and secrets outside of the source tree — use environment variables or a secret manager, not the JSON files in `Config/`.
- Restrict read access to the `Config/` directory to the service account only, since it may contain sensitive connection settings.

## 3. Transport Security (HTTPS)

- Call `UseHttpsRedirection()` in your middleware pipeline. In production, provision a certificate from a trusted CA (or your corporate PKI) — never a self-signed certificate.
- Terminate TLS at the application or at a reverse proxy in front of it and require TLS 1.2 as a minimum (TLS 1.3 recommended).
- Enable HSTS (`app.UseHsts()`) for browser-facing deployments.

## 4. Data at Rest

HTTPS protects data in transit. For sensitive data your application stores — database records, uploaded files, configuration exports — also consider encryption at rest.

- For PostgreSQL, review the available [encryption options](https://www.postgresql.org/docs/current/encryption-options.html) (filesystem-level, transparent data encryption, or column-level encryption via `pgcrypto`) and choose the approach that fits your threat model.
- Restrict filesystem access to the database data directory to the service account only.
- Avoid storing sensitive data in the MORYX `Config/` JSON files; prefer environment variables or a secret manager (see [Credentials and Secrets](#2-credentials-and-secrets)).

## 5. CORS

- Do not enable CORS in production unless a cross-origin client genuinely requires it. Any development CORS policy must be guarded by an environment check so it is never active in production.
- If CORS is required, restrict `WithOrigins` to the exact production origin — never combine `AllowCredentials` with a wildcard origin.

## 6. Static Files

ASP.NET Core's `UseStaticFiles` serves files from `wwwroot` by default, which is safe. Take care when reconfiguring the served path:

- Never point `UseStaticFiles` at directories that may contain secrets — `Config/`, log directories, or any path outside `wwwroot`.
- If you enable `UseDirectoryBrowser`, ensure it serves only a dedicated, non-sensitive directory and is protected by authentication — never point it at `wwwroot` root or any directory that may contain secrets.

See [Microsoft's security considerations for static files](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files#security-considerations-for-static-files) for details.

## 7. Logging

MORYX logging is built on `Microsoft.Extensions.Logging` (see [Logging](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/dev/docs/articles/framework/logging.md)). For production:

- Use a minimum level of `Warning`; enable `Debug` only temporarily for incident investigation.
- Never log credentials, tokens, personal data, or raw request bodies.
- Forward logs to a centralised, access-controlled and tamper-resistant sink with an appropriate retention period.

## 8. File Uploads (Media Module)

If your application uses the Media module, follow the [MORYX-Media Security Guidelines](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/dev/docs/articles/module-media/security-guidelines.md):

- Store uploaded files in a directory tree separate from the application.
- Remove execution privileges from the upload directory.

## 9. Network Hardening and Firewall

- Expose only the ports the application actually needs (typically 443 for HTTPS). Place MORYX behind a reverse proxy rather than exposing Kestrel directly to a public network.
- Keep the database reachable only from the application host.
- If OPC UA or MQTT integrations are used, ensure that communication with the corresponding servers or brokers is restricted to trusted industrial network segments.

## 10. Principle of Least Privilege

- Run the MORYX process under a dedicated service account without interactive login rights.
- Grant the database account only the permissions it needs on the application schema (no server-admin rights).
- The application requires write access only to its `Config/` and log directories; mount everything else read-only where possible.
- In container deployments, run as a non-root user.

## 11. Dependency Monitoring

MORYX centrally manages its dependencies, but you remain responsible for monitoring the dependencies of your own product for known CVEs:

- Watch [GitHub Security Advisories](https://github.com/PHOENIXCONTACT/MORYX-Framework/security/advisories) for MORYX-specific notifications.

### Third-Party MORYX Modules

By design, installed MORYX modules and their UIs can communicate freely within the application. This is a deliberate architectural feature, but it means that the security of a module you install is your responsibility:

- Use only modules from trustworthy vendors. Modules published directly by the MORYX team are maintained by PHOENIX CONTACT; modules from third-party sources require your own security assessment.
- Review the source, licensing, and update history of any third-party module before integrating it into a production application.

### Custom MORYX Modules

Modules you develop yourself — especially those with their own UI — introduce additional attack surface that is entirely your responsibility.

- Apply the same security practices to your module as to the host application (authentication, input validation, no hard-coded credentials).
- If your module exposes its own endpoints or UI, protect them with `[Authorize(Policy = …)]` just like any other controller.
- Familiarise yourself with the vulnerability classes relevant to your module's functionality (e.g. injection flaws for modules that process external data, XSS for modules with a web UI).

## 12. Supported Versions and End-of-Life

MORYX follows the [.NET LTS release schedule](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core).
Security updates are provided during both the **Active Support** and **Maintenance Support** phases.
For exact dates see the [MORYX Release Schedule](https://github.com/PHOENIXCONTACT/MORYX-Home/blob/main/processes/release-schedule.md).

| Version | Status              | Security updates until |
|---------|---------------------|------------------------|
| 10.x    | Active Support      | ~ Nov 2030             |
| 8.x     | Maintenance Support | ~ Nov 2028             |
| 6.x     | Maintenance Support | ~ Nov 2026             |
| < 6.0   | End of Life         | —                      |

Plan upgrades to a supported major version before its end-of-life date. Under CRA Article 13, the product you ship on top of MORYX must receive security updates for its expected lifetime (minimum five years) — factor MORYX's support window into your own support commitments.

## 13. Incident Response Contact

For a security incident in a product built on MORYX, contact the Phoenix Contact PSIRT:

- Web: [phoenixcontact.com/psirt](https://www.phoenixcontact.com/psirt)
- E-mail: [psirt@phoenixcontact.com](mailto:psirt@phoenixcontact.com)

For responsible disclosure of a vulnerability in the MORYX Framework itself, see [SECURITY.md](https://github.com/PHOENIXCONTACT/MORYX-Framework/blob/dev/SECURITY.md).
