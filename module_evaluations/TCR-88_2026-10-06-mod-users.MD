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

- **S005 — Personal-data disclosure:** The required personal-data disclosure file was found and parsed after the filename correction. Review of the code confirmed that user-related personal data is transmitted to Kafka. However, the disclosure’s overall accuracy and completeness, including its statements about free-form text, caching, and logging, still require manual review because the evaluator did not perform semantic review.
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

**Current status: Manual review required**

The automated evaluation found the personal-data disclosure file, but the semantic review was skipped. The disclosure therefore has not been fully verified.

**Disclosure selections reviewed**

The disclosure indicates that the module:
- Provides custom fields.
- Transmits personal data, including through queues and additional databases.
- Transmits free-form text fields.

The options stating that the module does not store personal data, does not process personal data, caches personal data, and logs personal data are unchecked.

**Evidence reviewed**

The `UserEventProducer` code sends user-created, user-deleted, and user-updated events to Kafka. The `UsersService.getConsortiumUserDto` method builds an event record containing user ID, username, barcode, external system ID, first and last name, email, phone, mobile phone, account status, and user type.

This evidence supports the disclosure’s statement that the module transmits personal data.

**Items requiring further verification**

- Confirm whether the disclosure accurately describes all personal-data categories handled by the module.
- Confirm whether other data flows transmit free-form text or custom-field values. The event-building method reviewed does not visibly include these fields, but other data flows have not been fully examined.
- Confirm that the unchecked caching and logging options accurately reflect the module’s behavior.
- Confirm that the disclosure is complete and accurate for the reviewed module version, `v19.6.2`.

**Conclusion**

The review provides evidence supporting the disclosure’s statement that the module transmits personal data. However, the disclosure’s overall accuracy and completeness remain unverified. S005 should remain subject to manual review until the outstanding questions are resolved.


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
| S005 — Personal-data disclosure | Manual review required. The disclosure file was located after the filename correction, and code review confirmed that user-related personal data is transmitted to Kafka. Further review would help confirm that the disclosure accurately and completely describes the relevant data flows, including free-form text, caching, and logging. |
| S010 — Runtime dependencies | Manual review required. Existing tests cover profile-picture operations, object storage, invalid inputs, and some configuration scenarios. Further review would help clarify behavior when storage settings are missing or invalid, the storage service fails or times out, Kafka is disabled, or producer shutdown fails. The inspected profile-picture retrieval method returns HTTP 404 for caught exceptions; confirm whether this response is appropriate for different types of storage errors. Remaining HTTP and database failure paths also warrant review. |
| API permissions | Manual review required. Permission declarations and selected endpoint mappings were inspected. Further review would help confirm runtime permission enforcement, field-level protection, and the completeness of the permission mappings. |
| API endpoint documentation | Preliminary; final coverage check outstanding. RAML documentation exists for the principal `mod-users` resources and the shared `folio-custom-fields` interface. The documentation for `/_/tenant` and `/_/tenant/{id}` remains to be confirmed. |
| Tenant data segregation | Manual review required. The inspected code selects tenant-specific PostgreSQL clients. Additional verification using distinct tenant IDs would help establish that tenant records remain isolated. No cross-tenant access violation has been established by the evidence reviewed. |
| Database schema access | No violation identified in the inspected evidence. A broader review of SQL scripts and database access paths would help confirm that the available evidence adequately addresses the evaluation criteria. |
| Dependency documentation | Manual review required. The descriptor lists `feesfines` and `configuration` as optional dependencies. Further review would help clarify the applicable documentation requirements and confirm whether the existing documentation addresses them. |
| Tenant-specific responses and `x-okapi-tenant` | Further verification required. Tenant-aware test setup was observed. Additional review would help confirm outgoing request-header behavior and isolation of responses between tenants. |
| `/admin/health` | Test coverage identified. `InstallUpgradeIT.java` contains an assertion for HTTP 200 and body `OK`. Confirming successful test execution in the applicable environment would complete this check. |
| High availability | Further verification required. The reviewed profile-picture storage path uses object storage. Further review of other relevant storage and session-related behavior, including multi-instance operation where practical, would help complete the module-wide assessment. |
| Supported infrastructure and platform technologies | Manual verification required. Java 21 aligns with the cited FOLIO Trillium guidance. Compliance of all relevant production and transitive dependencies remains unverified. |

These results summarize the evidence collected during the review. They are preliminary assessments, not declarations of complete compliance. Items marked unverified or requiring manual review should remain open until sufficient evidence has been collected.

## 18. Recommended follow-up actions

Prioritize the remaining work as follows:

1. **S005 — Personal-data disclosure:** Review the disclosure against the implementation and data flows. The review confirmed that user-related personal data is transmitted to Kafka. Verify the disclosure's accuracy and completeness, particularly the free-form text, caching, and logging selections, and confirm that the other selected personal-data categories are accurate.

2. **S010 — Runtime dependencies:** Review required and optional configuration behavior and clarify how the module handles outbox processing when Kafka is disabled. Existing tests cover profile-picture operations, including object storage, invalid inputs, and some configuration scenarios. Further review would help clarify behavior when object-storage settings are missing or invalid, or when the storage service encounters an error or timeout. The inspected profile-picture retrieval method returns HTTP 404 for caught exceptions; confirm whether this response is appropriate for different types of storage errors. The inspected Kafka producer code closes the producer when the shutdown operation succeeds. Further review would help clarify how cleanup is handled if shutdown fails and whether additional cleanup is performed elsewhere. Review the remaining HTTP and database failure paths to confirm that errors are handled as intended.

3. **API endpoint documentation:** Verify where `/_/tenant` and `/_/tenant/{id}` are documented. Compare the handlers listed in the module descriptor with the relevant RAML documentation to confirm that the documentation and declared endpoints are consistent before marking this criterion fully satisfied.

4. **API permissions:** Further review would help confirm how permissions are enforced at runtime, how field-level protection is applied, and whether the permission mappings are complete.

5. **Module descriptor and deployment descriptor:** Complete the endpoint-by-endpoint comparison to confirm that the permissions declared in the descriptors are consistent with the implementation.

6. **Reference data:** Verify installation and upgrade behavior, including how tenant-customized records are preserved, to confirm that the module behaves as intended across these scenarios.

7. **Integration tests:** Confirm which integration test suites pass in the applicable environment and verify whether the testing technologies meet the relevant official support requirements.

8. **Tenant isolation:** Review existing test coverage and, where needed, add tests using two distinct tenant IDs to help confirm that each tenant can access only its own records.

9. **`/admin/health`:** Confirm that the existing integration test passes in the applicable environment or, if appropriate, verify the endpoint in a deployed environment.

10. **Dependency documentation:** Confirm the applicable documentation requirements for the optional `feesfines` and `configuration` dependencies and whether the existing documentation addresses them adequately.

11. **High availability:** Review the remaining storage and session-related code to understand whether operation depends on local state or a particular application instance. Where practical, verify behavior when multiple instances are running.

12. **Supported technologies:** Compare the effective production dependencies, including transitive dependencies and resolved versions, with the official supported-technologies requirements. Include the S3-compatible storage library in the comparison to confirm that the relevant dependencies meet those requirements.

13. **Database schema access:** Extend the review of relevant SQL scripts and database access paths to confirm that the available evidence adequately addresses the evaluation criteria.

## 19. References

### Source code and module documentation

- [FOLIO mod-users v19.6.2 source code](https://github.com/folio-org/mod-users/tree/v19.6.2)
- [Module descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/ModuleDescriptor-template.json)
- [Deployment descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/DeploymentDescriptor-template.json)
- [Maven configuration (`pom.xml`)](https://github.com/folio-org/mod-users/blob/v19.6.2/pom.xml)
- [Module README](https://github.com/folio-org/mod-users/blob/v19.6.2/README.md)
- [Personal-data disclosure (`PERSONAL_DATA_DISCLOSURE.md`)](https://github.com/folio-org/mod-users/blob/v19.6.2/PERSONAL_DATA_DISCOSURE.md)
- [RAML API documentation directory](https://github.com/folio-org/mod-users/tree/v19.6.2/ramls)
- [Reference-data directory](https://github.com/folio-org/mod-users/tree/v19.6.2/src/main/resources)
- [Integration-test directory](https://github.com/folio-org/mod-users/tree/v19.6.2/src/test/java)
- [Install and upgrade integration tests](https://github.com/folio-org/mod-users/blob/v19.6.2/src/test/java/org/folio/rest/InstallUpgradeIT.java)

### Related FOLIO documentation

- [FOLIO Technical Council repository](https://github.com/folio-org/tech-council)
- [Shared FOLIO custom-fields repository](https://github.com/folio-org/folio-custom-fields)

**Review status:** Preliminary evaluation updated October 16, 2026. Targeted follow-up and manual verification remain outstanding.


