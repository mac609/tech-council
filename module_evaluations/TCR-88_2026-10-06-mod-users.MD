# Technical Council Module Evaluation: mod-users

- **Jira:** TCR-88
- **Module:** mod-users
- **Version reviewed:** v19.6.2
- **Evaluation date:** October 16, 2026
- **Repository evaluated:** FOLIO mod-users v19.6.2
- **Working evaluation document:** `TCR-88_2026-10-06-mod-users.MD`
- **Acceptance criteria:** FOLIO Technical Council repository

## 1. Executive summary

This document records a preliminary, evidence-based review of selected FOLIO Technical Council acceptance criteria for mod-users, a backend module that maintains user records and related user information.

The review considered source code, RAML API documentation, Maven configuration, module and deployment descriptors, integration-test files, the README, the personal-data disclosure, and the automated module-evaluation results.

The automated evaluator reported 31 criteria: 5 passed, 0 failed, and 26 requiring manual review after the personal-data disclosure filename was corrected. The evaluator found and parsed `PERSONAL_DATA_DISCLOSURE.md`, but its semantic review was skipped because agent review was disabled or unconfigured. The result therefore does not mean that all manual-review criteria have passed.

### Principal findings

- **S005 — Personal-data disclosure:** The required `PERSONAL_DATA_DISCLOSURE.md` file was found and parsed after the filename correction. Its substantive accuracy and completeness still require human review because the evaluator did not perform semantic review.
- **S010 — Runtime dependencies and configuration:** Several inspected database and Kafka write paths propagate failures through asynchronous futures. Configuration-absent or invalid behavior, object-storage error classification, Kafka-disabled outbox behavior, producer cleanup, and unreviewed failure paths still need verification.
- **API permissions:** The module descriptor declares permissions for the inspected handlers. Complete permission coverage, runtime authorization, and field-level protection remain unverified.
- **API endpoint documentation:** The module uses RAML documentation. The checklist explicitly permits RAML for existing RMB-based modules. RAML files were reviewed for the principal mod-users API resources. The separate folio-custom-fields repository also contains RAML documentation for custom-field collection and item operations and both custom-field statistics endpoints. A complete comparison of all descriptor-declared endpoints has not been completed; the `/_/tenant` and `/_/tenant/{id}` paths remain to be checked.
- **Module descriptor and reference data:** The reviewed descriptor and implementation provide evidence of alignment. Reference-data loading is version-aware in the inspected code. Complete descriptor comparison and actual installation/upgrade behavior remain unverified.
- **Integration tests:** Integration-test classes and test dependencies are present. The presence of tests does not establish that the complete suite passes.
- **Tenant isolation and tenant-specific responses:** The inspected code uses tenant-aware PostgreSQL client selection. Cross-tenant isolation and outgoing request-header behavior have not been fully demonstrated.
- **Database schema and dependency documentation:** No prohibited cross-schema access was identified in the limited search reviewed. The applicable documentation requirements for optional dependencies need clarification.
- **Health endpoint:** `InstallUpgradeIT.java` contains a test asserting that `GET /admin/health` returns HTTP 200 and body `"OK"`. Successful execution of the test remains unverified.
- **High availability and supported technologies:** The reviewed profile-picture storage path uses an object-storage client. The Maven configuration specifies Java 21, consistent with the cited FOLIO Trillium backend technology guidance. Module-wide high-availability behavior and full production dependency compliance remain unverified.

**Overall conclusion:** This is a preliminary evaluation, not a declaration that all acceptance criteria have passed. Findings marked for manual review or further verification should remain open until supported by sufficient evidence.

## 2. Review scope and method

The review covered:

- Automated evaluation results, particularly S005 and S010.
- API permissions and module descriptor configuration.
- API endpoint documentation, including RAML resources and the shared custom-fields interface.
- Reference data and integration testing.
- Tenant-aware database access and database schema usage.
- Dependency documentation.
- Tenant-specific request handling.
- The `/admin/health` endpoint.
- High-availability considerations.
- Infrastructure and platform technologies.

Evidence was collected by inspecting the relevant source files and configuration. The review distinguishes between evidence that a file or declaration exists and evidence that behavior has been tested. Where the available evidence did not establish runtime behavior or complete coverage, the finding remains preliminary.

## 3. S005 — Personal-data disclosure

**Assessment:** Manual review required; semantic compliance has not been verified.

**Evidence**

The automated evaluator found and parsed `PERSONAL_DATA_DISCLOSURE.md` after the filename was corrected. The latest reported evaluator run showed 31 criteria, 5 passed, 0 failed, and 26 requiring manual review.

The evaluator output states that semantic review was skipped because agent review was disabled or unconfigured.

Finding and parsing the file establishes that the required artifact is present and readable. It does not establish that the disclosure accurately describes all relevant personal-data collection, processing, storage, transmission, sharing, and retention.

The disclosure indicates that the module provides custom fields and identifies categories of personal information. Those selections should be compared with the implementation and actual data flows. User-event code and other paths that transmit user information deserve particular attention.

**Finding**

The disclosure artifact is present and parseable, but its substantive accuracy and completeness have not been verified.

**Recommended action**

Compare `PERSONAL_DATA_DISCLOSURE.md` with the implementation and relevant data flows. Confirm that the selected data categories and statements about storage, processing, transmission, and other handling practices are accurate before marking S005 as satisfied.

**Status:** Manual review required.

## 4. S010 — Runtime dependencies and configuration

**Assessment:** Manual review required. The inspected code demonstrates explicit failure propagation in several database and Kafka write paths. Runtime behavior is not fully verified for all dependencies and configuration states.

**Evidence reviewed**

**1. Configuration declarations**

`descriptors/ModuleDescriptor-template.json` declares configuration inputs for database connectivity, Kafka, and object storage. The descriptor alone does not establish which settings are mandatory at runtime or what happens when settings are missing or invalid.

**2. HTTP client behavior**

`HttpClientFactory.java` creates and caches a `VertxOkapiHttpClient` per Vert.x instance. `VertxOkapiHttpClient.java` applies explicit timeouts to several request methods and converts responses to the wrapper's response model. Failure behavior across all HTTP operations has not been fully verified.

**3. HTTP response handling**

`ResponseInterpreter.java` converts unsuccessful HTTP responses into `HttpException` failures. `FeesFinesModuleClientImpl.java` logs and propagates failures in the inspected operations.

**4. Database failures**

`RecordRepositoryImpl.java` uses `getAsyncResultHandler` for inspected read and update operations. `ResultHandlerUtils.java` completes a promise successfully when an asynchronous operation succeeds and fails it with the original cause when the operation fails. This supports failure propagation in the reviewed repository paths, but does not establish how every database failure is ultimately returned to API callers.

**5. Object-storage failures**

`ProfilePictureStorage.java` catches exceptions in inspected object-storage operations and translates them into API responses. Some caught failures, including failures retrieving a presigned URL or fetching an image, can be returned as 404. Further review is needed to determine whether storage outages and other infrastructure failures are classified appropriately and whether timeout behavior is suitable.

**6. Kafka write failures**

`UserEventProducer.java` checks the result of `producer.write(...)`. On failure, it logs the error and fails the returned future with the original cause. The inspected `getKafkaFutures(...)` and `processOutboxEventLogs(...)` methods compose futures before attempting outbox deletion. Under standard Vert.x future-composition behavior, a failed send should fail this chain and prevent the subsequent deletion step.

**7. Kafka-disabled behavior**

The default `UserEventProducer` constructor obtains its enablement setting from `KafkaConfigSingleton.INSTANCE.isKafkaEnabled()`. When publishing is disabled, `sendUserEvent(...)` returns a successful future containing `false`. The inspected outbox-processing code does not inspect individual Boolean results before deriving event IDs for deletion. The intended behavior, including whether outbox records should be retained when Kafka is disabled, remains unverified.

**8. Producer cleanup**

`sendToKafka(...)` calls `producer.end()` after the write completes and calls `producer.close()` on successful completion of `end()`. The inspected code does not establish how cleanup is handled if `end()` fails.

**Finding**

The inspected paths provide evidence of explicit failure propagation for database operations and Kafka writes. They do not establish complete runtime resilience across all dependencies or configuration states.

**Recommended follow-up**

- Verify required and optional configuration settings and behavior when settings are missing or invalid.
- Confirm the intended outbox behavior when Kafka publishing is disabled, including whether events should remain available for later delivery.
- Verify object-storage error classification and timeout behavior.
- Confirm producer cleanup when `end()` fails.
- Review remaining database and HTTP client failure paths, including how failures are surfaced to API callers.

**Status:** Manual review required.

## 5. API permissions

**Assessment:** Manual review required.

**Evidence**

The inspected `descriptors/ModuleDescriptor-template.json` defines permission sets and API handler mappings through `permissionsRequired`, `permissionsDesired`, and selected `modulePermissions` entries.

The inspected mappings cover:

- User records
- Profile pictures
- User groups, address types, proxies, and departments
- Custom fields, including statistics
- Patron PINs
- User settings

The reviewed `permissionsRequired` entries correspond to permission names declared in `permissionSets`.

The composite permission `users.all` includes permissions for user records and related interfaces, including groups, address types, proxies, departments, patron PINs, tenant affiliations, profile pictures, and user settings.

The composite permission `user-settings.custom-fields.all` groups eight custom-field permissions.

The `/users` and `/users/{id}` handlers declare `users.basic-read.execute` and `users.restricted-read.execute` under `permissionsDesired`, alongside the permissions required for collection and individual-item operations. The presence of these declarations does not, by itself, confirm how basic and restricted field-level access is enforced at runtime.

The user-delete handler also declares `manualblocks.collection.get` and `manualblocks.item.delete` in `modulePermissions`. The purpose and runtime enforcement of these entries require further review.

The RAML validation trait includes a `422` response for validation errors. This documents validation behavior, but does not establish that authorization is correctly enforced.

**Finding**

Permission declarations and selected endpoint mappings were inspected. Complete authorization behavior, field-level protection, and the completeness and purpose of all permission mappings have not been verified.

**Recommended action**

- Verify that permissions are enforced at runtime for the relevant endpoints.
- Confirm how basic and restricted read permissions control access to user fields.
- Check that all declared endpoints have appropriate permission mappings.
- Verify the purpose and enforcement of the `modulePermissions` entries.
- Confirm that composite permissions include the intended permissions and scope.

**Status:** Manual review required.

## 6. API endpoint documentation

**Assessment:** Preliminary; a complete endpoint documentation check remains outstanding.

**Evidence**

The Technical Council criterion calls for API endpoints to be documented in OpenAPI. Existing RMB modules may continue to use RAML under the stated exception. `mod-users` uses RAML documentation.

The inspected RAML files include:

- `users.raml`
- `groups.raml`
- `addressTypes.raml`
- `proxiesFor.raml`
- `departments.raml`
- `patronpin.raml`
- `userTenants.raml`
- `settings.raml`
- `staging_users.raml`

`users.raml` documents user collection and individual-user operations. It also documents related operations for expiration timers, outbox processing, profile-picture operations and cleanup, and configuration entries.

The shared `raml-util/rtypes/item-collection.raml` file defines standard GET, PUT, and DELETE operations for individual records.

**Documentation issue identified**

The title in `departments.raml` says “User Groups,” although the file documents the departments resource. This is a documentation inconsistency that should be corrected or clarified.

**Custom-fields endpoints**

`ModuleDescriptor-template.json` declares handlers for the following custom-fields endpoints:

- `/custom-fields`
- `/custom-fields/{id}`
- `/custom-fields/{id}/stats`
- `/custom-fields/{id}/options/{optId}/stats`

The `mod-users` repository does not contain a `custom-fields.raml` file. However, the shared `folio-org/folio-custom-fields` repository contains `ramls/custom-fields.raml`, which documents collection, item, and statistics endpoints.

**Tenant endpoints**

The module descriptor declares the `/_tenant` interface with the `/_/tenant` and `/_/tenant/{id}` endpoints. Documentation for these endpoints has not yet been confirmed.

**Finding**

RAML documentation exists for the principal `mod-users` resources, and shared documentation exists for the custom-fields interface. A complete comparison of all declared handlers against the applicable API documentation has not been completed.

**Recommended action**

- Confirm where `/_/tenant` and `/_/tenant/{id}` are documented.
- Compare all API handlers in the module descriptor against the relevant RAML documentation.
- Clarify or correct the title in `departments.raml`.
- Confirm that the applicable RAML exception covers this module.

**Status:** Preliminary; final documentation coverage check outstanding.

## 7. Module descriptor and deployment descriptor

**Assessment:** Manual review required.

**Evidence**

The review examined the following files:

- `descriptors/ModuleDescriptor-template.json`
- `descriptors/DeploymentDescriptor-template.json`
- `UsersAPI.java`

The module descriptor identifies the module version, interfaces, API handlers, permissions, optional dependencies, and scheduled jobs.

The deployment descriptor specifies the Java launch command used to start the module.

`UsersAPI.java` implements core user operations and includes functionality related to profile-picture management, configuration, expiration, and outbox processing.

The module descriptor declares the custom-fields collection PUT permission as `user-settings.custom-fields.collection.put`. This permission is declared in `permissionSets` and included in the composite permission `user-settings.custom-fields.all`.

User-settings handlers also use individual permissions grouped under `users.settings.all`.

**Finding**

The inspected descriptors and implementation provide evidence of alignment between selected API operations and their declarations. However, a complete comparison of every endpoint, its required permissions, and its implementation has not been performed.

**Recommended action**

- Compare all declared API handlers with the corresponding implementation.
- Verify that each endpoint has the appropriate permission declarations.
- Confirm that the declared permissions match the actual runtime authorization behavior.

**Status:** Manual review required.

## 8. Reference data

**Assessment:** Manual review required.

**Evidence**

The inspected reference-data files define the following address types:

- Claim
- Home
- Order
- Payment
- Returns
- Work

The user groups include:

- Faculty
- Graduate
- Staff
- Undergraduate

`TenantRefAPI.loadData()` conditionally loads reference data using the `isNew()` check. The address types data is associated with version `15.4.0`, and the groups data is associated with version `17.3.0`.

The `isNew()` logic treats a null `moduleFrom` value as a new installation. Otherwise, it uses `Versioned.isNewForThisInstall()` to determine whether the data should be loaded for the current installation or upgrade.

The `MODUSERS-335` change was intended to prevent unchanged reference or sample data from being restored during tenant upgrades.

**Finding**

The inspected implementation provides evidence that reference-data loading considers installation and version information. The actual behavior during installation and upgrades has not been verified, including whether tenant-customized records are preserved.

**Recommended action**

- Verify reference-data loading during a new tenant installation.
- Verify behavior during upgrades between module versions.
- Confirm that tenant-customized reference records are not overwritten or restored unexpectedly.

**Status:** Manual review required.

## 9. Integration tests and testing technology

**Assessment:** Further verification required.

**Evidence**

The review identified the following integration-test classes:

- `AbstractRestTest.java`
- `AbstractRestTestNoData.java`
- `AbstractRestTestWithData.java`
- `ExpirationIT.java`
- `GroupIT.java`
- `ProxyRelationshipsIT.java`
- `UserOutboxProcessIT.java`

The Maven configuration declares the following testing dependencies:

- JUnit BOM `6.0.3`
- JUnit Jupiter API, Engine, and Params
- Testcontainers JUnit Jupiter
- Testcontainers LocalStack
- JUnit Vintage Engine

`UserOutboxProcessIT.java` includes tests for successful Kafka user-event processing and an error involving a nonexistent tenant.

**Finding**

Integration-test classes and testing dependencies are present. Their presence does not establish that the tests pass or that the testing technologies meet the applicable official support requirements. Successful execution of the relevant test suite has not been verified.

**Recommended action**

- Confirm whether the relevant integration-test suite passes in the intended environment.
- Review test failures, if any, and determine whether they affect the evaluation.
- Verify that the testing technologies and versions meet the applicable FOLIO support requirements.

**Status:** Further verification required.

## 10. Tenant data segregation at the storage layer

**Assessment:** Manual review required.

**Evidence**

The review examined `RecordRepositoryImpl.java`, `UserOutboxService.java`, and `ProfilePictureStorage.java` for evidence of tenant-aware data access.

`RecordRepositoryImpl.java` uses tenant-specific PostgreSQL clients for the inspected retrieval and update operations. The reviewed code includes the following method:

`private PostgresClient pgClient(String tenantId) { return PostgresClient.getInstance(vertx, tenantId); }`

`UserOutboxService.java` obtains the tenant ID from the Okapi request headers.

`AbstractRestTest.java` uses the tenant ID `diku` in both Okapi headers and the PostgreSQL client setup.

**Finding**

The inspected code provides evidence that these paths use tenant-specific database clients or tenant information. However, this does not demonstrate that all data-access paths prevent one tenant from accessing another tenant's records.

No explicit test using two different tenants to verify cross-tenant isolation was identified in the reviewed evidence. No tenant-isolation violation has been established.

**Recommended action**

- Review remaining database and storage access paths for tenant-specific handling.
- Review existing tests for cross-tenant access protection.
- Where practical, add or run tests using two distinct tenant IDs to verify that one tenant cannot access another tenant's data.

**Status:** Manual review required.

## 11. Database schema access

**Assessment:** No violation identified in the inspected evidence; broader verification remains outstanding.

**Evidence**

A search for the literal text `public.` found one result in `create_configuration_table.sql`. The result references `public.uuid_generate_v4()` in a function or default-value expression.

The inspected table definition uses the `configuration` table without a different schema prefix.

A search of `RecordRepositoryImpl.java` did not identify any `public.` matches.

**Finding**

The limited searches did not identify evidence of an application table being accessed through an unintended schema. However, these searches do not establish that every SQL script and database access path follows the applicable schema requirements.

**Recommended action**

- Review the remaining SQL scripts and database access paths.
- Confirm that schema references follow the applicable FOLIO database requirements.
- Investigate any additional schema-qualified table references found during the broader review.

**Status:** No violation identified in the inspected evidence; further verification required.

## 12. Dependency documentation

**Assessment:** Manual review required.

**Evidence**

The module descriptor lists `feesfines` and `configuration` as optional dependencies.

The README describes profile-picture and S3-related functionality, but the reviewed material does not clearly document the `feesfines` dependency.

The applicable documentation requirements for optional dependencies have not been confirmed.

**Finding**

The module descriptor identifies the optional dependencies, but the available evidence does not establish whether their documentation meets all applicable requirements. No confirmed violation has been identified.

**Recommended action**

- Confirm the Technical Council's documentation requirements for optional dependencies.
- Check whether the README or other module documentation should explain the purpose of `feesfines` and `configuration`.
- Update the documentation if required.

**Status:** Manual review required.

## 13. Tenant-specific responses and `x-okapi-tenant`

**Assessment:** Further verification required.

**Evidence**

`AbstractRestTest.java` sets the tenant name in the Okapi request headers using `OkapiHeaders(wireMockUrl, TENANT_NAME, token)`.

The test setup also creates a PostgreSQL client using `PostgresClient.getInstance(vertx, TENANT_NAME)` and passes the headers into module initialization.

This demonstrates tenant-aware test setup for the inspected paths. It does not establish that every outgoing request carries the correct `x-okapi-tenant` header or that responses are isolated correctly between tenants.

**Finding**

The inspected test setup uses tenant-specific information, but outgoing request-header behavior and cross-tenant response isolation remain unverified. No violation has been established by the reviewed evidence.

**Recommended action**

- Review outgoing requests to confirm that the correct `x-okapi-tenant` header is used where required.
- Verify that responses and stored data are isolated between tenants.
- Review or add tests that exercise requests using two different tenant IDs.

**Status:** Further verification required.

## 14. `/admin/health` endpoint

**Assessment:** Test coverage identified; successful execution remains unverified.

**Evidence**

The integration test `src/test/java/org/folio/rest/InstallUpgradeIT.java` contains a health-check test that sends a GET request to `/admin/health`.

The test expects an HTTP `200` response and a response body containing `OK`.

**Finding**

The test provides evidence that the module has integration-test coverage for the `/admin/health` endpoint. The available evidence does not establish that the test has passed or that the endpoint has been verified in a deployed environment.

**Recommended action**

- Confirm that the health-check integration test passes in the applicable test environment.
- If needed, verify that `/admin/health` returns the expected response in a deployed environment.

**Status:** Test coverage identified; successful execution remains unverified.

## 15. High availability

**Assessment:** Further verification required.

**Evidence**

`ProfilePictureStorage.java` obtains the tenant ID from the `x-okapi-tenant` header and uses it to prefix object-storage keys. The inspected code uses `FolioS3Client` for profile-picture storage and handles image data through in-memory byte streams.

A search of the reviewed `ProfilePictureStorage.java` file did not identify a `FileOutputStream` match.

**Finding**

The inspected profile-picture storage path uses object storage rather than an identified local file-output stream in that file. This does not establish that the entire module avoids local state or session affinity, nor does it demonstrate that the module behaves correctly when multiple instances are running.

**Recommended action**

- Review the remaining module code for local state or other instance-specific dependencies.
- Verify whether any session affinity is required.
- Where practical, test module behavior with multiple instances.

**Status:** Further verification required.

## 16. Officially supported infrastructure and platform technologies

**Assessment:** Manual verification required.

**Evidence**

The review identified the following technologies and dependencies in the module's Maven configuration and source code.

| Technology or component | Evidence |
|---|---|
| Java | `pom.xml` sets Java version 21. The README lists Java 21 as a JDK prerequisite. |
| Vert.x | The `vertx-stack-depchain` dependency is version `5.0.10`. |
| PostgreSQL testing | `org.folio:postgres-testing` is declared with test scope. Its version is managed by `${raml-module-builder.version}`. |
| FOLIO Kafka wrapper | `org.folio:folio-kafka-wrapper` is version `4.0.0`. |
| Apache Kafka client | `org.apache.kafka:kafka-clients` is version `4.2.0`. |
| S3-compatible storage | `FolioS3ClientFactory` and `FolioS3Client` are used in the profile-picture storage implementation. |

The reviewed FOLIO Trillium guidance specifies Java 21/JDK 21, which aligns with the Java version declared by the module.

A search for `aws` in `pom.xml` did not identify a matching dependency. This search alone does not establish whether the storage implementation's required libraries are absent, because they may use different names or be supplied transitively.

**Finding**

The declared Java version aligns with the cited FOLIO Trillium guidance. The available evidence does not establish that all production dependencies, transitive dependencies, and resolved versions comply with the applicable supported-technology requirements.

**Recommended action**

- Compare the effective production dependency tree, including transitive dependencies and resolved versions, with the official supported-technology requirements.
- Confirm the versions of dependencies managed through the Maven configuration.
- Verify which library provides the S3-compatible storage functionality and whether it meets the applicable requirements.

**Status:** Manual verification required.

## 17. Consolidated status

| Criterion | Preliminary result |
|---|---|
| S005 — Personal-data disclosure | Manual review required. The evaluator found and parsed `PERSONAL_DATA_DISCLOSURE.md`, but skipped semantic review. The disclosure's accuracy and completeness remain unverified. |
| S010 — Runtime dependencies | Manual review required. Several inspected database and Kafka write paths propagate failures. Configuration, object-storage errors, Kafka-disabled outbox processing, producer cleanup, and other failure paths need further review. |
| API permissions | Manual review required. Permission declarations and selected endpoint mappings were inspected. Complete authorization and field-level protection remain unverified. |
| API endpoint documentation | Preliminary; final coverage check outstanding. RAML documentation exists for the principal `mod-users` resources and the shared `folio-custom-fields` interface. Documentation for `/_/tenant` and `/_/tenant/{id}` has not been confirmed. |
| Module descriptor and deployment descriptor | Manual review required. The inspected descriptor and implementation show evidence of alignment, but a complete endpoint-by-endpoint comparison has not been performed. |
| Reference data | Further verification required. The inspected implementation supports version-aware loading. Actual installation, upgrade, and preservation of tenant-customized records remain unverified. |
| Integration tests and testing technology | Further verification required. Integration-test classes and dependencies are present. Successful execution of the full test suite and official technology support remain unverified. |
| Tenant data segregation | Manual review required. Inspected code selects tenant-specific PostgreSQL clients. Cross-tenant isolation has not been demonstrated; no violation has been established. |
| Database schema access | No violation identified in the inspected evidence. Broader verification of SQL scripts and database operations remains outstanding. |
| Dependency documentation | Manual review required. The descriptor lists `feesfines` and `configuration` as optional dependencies. Applicable documentation requirements need clarification. |
| Tenant-specific responses and `x-okapi-tenant` | Further verification required. Tenant-aware test setup was observed, but outgoing request-header behavior and cross-tenant response isolation remain unverified. |
| `/admin/health` | Test coverage identified. `InstallUpgradeIT.java` asserts HTTP 200 and body `OK`; successful execution remains unverified. |
| High availability | Further verification required. The reviewed profile-picture storage path uses object storage, but module-wide high-availability behavior remains unverified. |
| Supported infrastructure and platform technologies | Manual verification required. Java 21 aligns with the cited FOLIO Trillium guidance. Compliance of all relevant production and transitive dependencies remains unverified. |

These results summarize the evidence collected during the review. They are preliminary assessments, not declarations of complete compliance. Items marked unverified or requiring manual review should remain open until sufficient evidence has been collected.

## 18. Recommended follow-up actions

Prioritize the remaining work as follows:

1. **S005 — Personal-data disclosure:** Manually review `PERSONAL_DATA_DISCLOSURE.md` against the implementation and data flows. Confirm that the disclosure is accurate and complete.

2. **S010 — Runtime dependencies:** Verify required and optional configuration behavior; confirm Kafka-disabled outbox handling; review object-storage error classification and timeout behavior; verify producer cleanup when `end()` fails; and inspect remaining HTTP and database failure paths.

3. **API endpoint documentation:** Confirm where `/_/tenant` and `/_/tenant/{id}` are documented. Compare the module descriptor's declared handlers against the relevant RAML documentation before marking this criterion fully satisfied.

4. **API permissions:** Verify runtime enforcement, field-level protection, and completeness of permission mappings.

5. **Module descriptor and deployment descriptor:** Complete the endpoint-by-endpoint comparison and verify that declared permissions align with the implementation.

6. **Reference data:** Verify installation and upgrade behavior, including preservation of tenant-customized records.

7. **Integration tests:** Confirm which test suites pass and verify the applicable official support status of the testing technologies.

8. **Tenant isolation:** Review or add tests using two distinct tenant IDs to confirm that one tenant cannot access another tenant's records.

9. **`/admin/health`:** Confirm that the integration test passes in the applicable environment or verify the endpoint in a deployed environment.

10. **Dependency documentation:** Confirm the applicable documentation requirements for the optional `feesfines` and `configuration` dependencies.

11. **High availability:** Review remaining storage and session-related code for local state or instance affinity, and test multi-instance behavior where practical.

12. **Supported technologies:** Compare the effective production dependency tree, including transitive dependencies and resolved versions, with the official supported-technologies requirements. Verify the S3-compatible storage library as well.

13. **Database schema access:** Complete a broader review of SQL scripts and database access paths.

## 19. References

The evaluation used the following source materials:

- FOLIO `mod-users` version `v19.6.2` source code.
- FOLIO Technical Council module acceptance criteria.
- `descriptors/ModuleDescriptor-template.json`.
- `descriptors/DeploymentDescriptor-template.json`.
- Maven configuration (`pom.xml`).
- Module README.
- `PERSONAL_DATA_DISCLOSURE.md`.
- RAML documentation in the `mod-users` repository.
- Module descriptor from the reviewed `mod-users` fork snapshot.
- Shared `folio-custom-fields` RAML documentation.
- Reference-data files and implementation.
- Integration-test classes.
- `src/test/java/org/folio/rest/InstallUpgradeIT.java`, including the `/admin/health` test.

**Review status:** Preliminary evaluation updated October 16, 2026. Targeted follow-up and manual verification remain outstanding.

