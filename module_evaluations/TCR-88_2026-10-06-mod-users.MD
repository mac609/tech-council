# TCR-88 — mod-users Evaluation

**Evaluation date:** October 6, 2026

# FOLIO Module Acceptance Values and Criteria: mod-users

## version 3.0 (ratified 2025-01-25) + draft notes for existing module evaluation

## Overview
The Technical Council (TC) has defined a process for technical evaluation of modules for inclusion in a FOLIO release.  This document outlines the high level values and specific criteria used when evaluating modules as part of this process.  The distinction between values and criteria is as follows:

* **Values** are high-level factors that are unlikely to change over time.  They should be understandable even to a non-technical audience.
* **Criteria** are specific, verifiable tests as to how we’re meeting the Values within the current state of affairs.  Criteria will evolve over time to respond to changes in the Project’s technology stack, current best practices, etc. They may require a degree of technical expertise or domain knowledge to fully understand.

In some cases, we may not yet have comprehensive Criteria for assessing a module’s adherence to a Value.  In such cases, subjective review and analysis will be applied, and may lead to discussion.

This document provides the criteria against which a module will be assessed for inclusion into a FOLIO release, following the [New Module Technical Evaluation Process](NEW_MODULE_TECH_EVAL.MD). 

Existing FOLIO modules ideally conform to the same Values and Criteria.  It is understood that not all existing modules currently do so, especially modules created before the Values & Criteria were initially defined.  The Technical Council will work with development teams to align the current reality of the code with the Values and Criteria over time, as practicable.  Such processes will need to be developed and documented.  The Technical Council is currently piloting an existing module evaluation process with a few initial modules, and has noted below with "For existing module evaluation" where individual criteria will be handled differently.

Please see [Before Development](NEW_MODULE_TECH_EVAL.MD#before-development) for any planned development to new or existing modules that may intentionally diverge from these Values or Criteria.

## How to use this document

When performing a technical evaluation of a module, create a copy of this document.  The evaluation results should be placed in the [module_evaluations](https://github.com/folio-org/tech-council/tree/master/module_evaluations) directory and should conform to the following naming convention: `{JIRA Key}_YYYY-MM-DD-{module name}.MD`, e.g. `TCR-1_2021-11-17-mod-foo.MD`.  The date here is used to differentiate between initial and potential re-evaluation(s).  It should be the date when the evaluation results file was created.

Replace the title with the name of the module or library.

Remove either the Frontend or the Backend section.

Use these conventions to indicate the status of each criterion.

* [x] ACCEPTABLE
* [x] ~INAPPLICABLE~
* [ ] UNACCEPTABLE
  * comments on what was evaluated/not evaluated, why a criterion failed

## Values
1. Module adheres to Community Code of Conduct
2. Module license is compatible with the FOLIO Project
3. Module can be included in existing community build processes
4. Module has robust testing that can run with existing community testing processes
5. Module can be deployed in the Community’s reference environments without undue burden
6. Module is secure
7. Module is multi-tenant
8. Module is internationalized
9. Module meets current accessibility requirements
10. Module offers a cohesive user experience consistent with the rest of FOLIO
11. Module has developer and end-user documentation
12. Module depends ONLY on other modules and infrastructure that are already included in FOLIO
13. Module has a long-term development and maintenance plan
14. Module is scalable
15. Module supports high availability
16. Modules conforms to FOLIO upgrade mechanisms

## Criteria

### Administrative
* [x] ~INAPPLICABLE~ Listed by the Product Council on [Functionality Evaluated by the PC](https://folio-org.atlassian.net/wiki/spaces/PC/pages/4885367/Functionality+Evaluated+by+the+PC) with a positive evaluation result.
  - For existing module evaluation: This criterion is inapplicable.

### Shared/Common
* [x] ACCEPTABLE Uses Apache 2.0 license (2)
  - The automated evaluator reported this criterion as passed for the reviewed evaluation source.
* [x] ACCEPTABLE Module build MUST produce a valid module descriptor (3, 5)
  * _This is not applicable to libraries_
  - `mvn process-resources -DskipTests` generated `target/ModuleDescriptor.json`, and the generated descriptor passed Okapi schema validation. This supports the build criterion; it does not by itself establish that the descriptor committed in the repository meets every backend descriptor requirement.
* [ ] Inclusion of third party dependencies complies with [ASF 3rd Party License Policy](https://apache.org/legal/resolved.html) (2)
  * Uses README for [Category B Appropriately Labelled Condition](https://apache.org/legal/resolved.html#appropriately-labelled-condition)
  * LGPL consideration:
    * org.z3950.zing:cql-java is allowed if appropriately labelled, even if it is LGPL-2.1-only
    * org.marc4j:marc4j is allowed if appropriately labelled, even if it is LGPL-2.1-or-later
    * org.hibernate.* is allowed if appropriately labelled, even if it is LGPL-2.1-or-later
    * By [request of the Community Council](https://folio-org.atlassian.net/wiki/spaces/CC/pages/1243348996/2025-09-26+Community+Council+Meeting+Minutes+at+WOLFcon), and until that request changes, no additional LGPL-licensed dependencies will be allowed. 
  * _note: If a library declares multiple licenses in its pom.xml, [only one of them needs to comply](https://maven.apache.org/ref/3.9.11/maven-model/maven.html#project)._ (This applies only to Maven. For other package managers, closer evaluation is needed.)
  * _note: The [FOLIO Module Evaluator](https://github.com/folio-org/tc-module-eval) tool is optionally available to evaluate this criterion for Java modules (built with Maven) and for front-end modules.  TC expects to [require its use](https://github.com/folio-org/tech-council/pull/110) in the module evaluation process after a trial period._
  * _note: [More information about this criterion](https://github.com/folio-org/tech-council/blob/master/criteria/THIRD_PARTY_DEPENDENCIES.MD)_
  - The dependency inventory flagged `org.z3950.zing:cql-java` as LGPL-licensed. Please verify that the [README](https://github.com/folio-org/mod-users/blob/v19.6.2/README.md) includes the required Category B labelling and that the other dependencies meet the policy. The [Maven configuration](https://github.com/folio-org/mod-users/blob/v19.6.2/pom.xml) and inventory should be checked together; the inventory alone does not establish a policy violation.
* [ ] Installation documentation is included (11)
  * -_note: read more at https://github.com/folio-org/mod-search/blob/master/README.md_
  * _This is not applicable to libraries_
  - The [README](https://github.com/folio-org/mod-users/blob/v19.6.2/README.md) is present. The completeness of installation, configuration, and operating instructions has not been fully assessed.
* [ ] [Personal data form](https://github.com/folio-org/personal-data-disclosure) is completed, accurate, and provided as PERSONAL_DATA_DISCLOSURE.md file (6)
  * _This is not applicable to libraries_
  - The original automated evaluation failed this criterion because the expected file `PERSONAL_DATA_DISCLOSURE.md` was missing; the source repository at the reviewed tag contains the misspelled file [PERSONAL_DATA_DISCOSURE.md](https://github.com/folio-org/mod-users/blob/v19.6.2/PERSONAL_DATA_DISCOSURE.md). The original run reported 31 criteria: 5 passed, 1 failed, and 25 requiring manual review. Code review of [UserEventProducer.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/event/service/UserEventProducer.java) and [UsersService.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/service/UsersService.java) found user-related personal data transmitted in Kafka events and included in consortium-user data. Please verify that the disclosure accurately covers the fields and data flows, including free-form text, caching, and logging.
* [ ] Sensitive and environment-specific information is not checked into git repository (6)
  - No confirmed exposure was identified in the material reviewed so far. The review scope is the [v19.6.2 source repository](https://github.com/folio-org/mod-users/tree/v19.6.2); a complete repository-wide check has not been documented.
* [ ] There is no default for any password, access key, private key or any other credential (6)
  - No default credential was confirmed in the reviewed material. Further verification of configuration and startup paths is needed before concluding that the criterion is met.
* [ ] Written in a language and framework from the [officially supported technologies](https://folio-org.atlassian.net/wiki/spaces/TC/pages/5053681/Officially+Supported+Technologies) page[^1] (3, 5)
  - The evaluator reported this criterion as passed. The exact source revision used by the evaluator has not yet been reconciled with the `v19.6.2` tag, so confirm the result against the reviewed release.
* [ ] Uses FOLIO interfaces already provided by previously accepted modules _e.g. a UI module cannot be accepted that relies on an interface only provided by a back end module that hasn’t been accepted yet_ (3, 5, 12)
  * _This is not applicable to libraries_
  - The evaluator reported this criterion as passed. Verify that its input source matches the `v19.6.2` release being evaluated.
* [ ] Must not depend on a FOLIO library that has not been approved through the TCR process
  - The evaluator reported this criterion as passed. Verify that its input source matches the `v19.6.2` release being evaluated.
* [ ] Sonarqube hasn't identified any security issues, any high or greater severity issues, or excessive (>3%) duplication (6); and any disabled or intentionally ignored rules/recommendations are reasonably justified.
  * See [Rule Customization](https://dev.folio.org/guides/code-analysis/#rule-customization) details. 
  - The available SonarQube snapshot reported no security issues, no security hotspots, and 0% duplication, with 26 other issues. The snapshot was labelled `19.7.0-SNAPSHOT`, rather than `v19.6.2`; verify the scan for the exact release and whether any rules were disabled or issues intentionally ignored. The release's [Maven configuration](https://github.com/folio-org/mod-users/blob/v19.6.2/pom.xml) is the version reference for follow-up.
* [ ] Uses [officially supported](https://folio-org.atlassian.net/wiki/spaces/TC/pages/5053681/Officially+Supported+Technologies) build tools (3, 5, 13)
  - The evaluator reported this criterion as passed. Confirm the evaluation source and tool versions correspond to `v19.6.2`.
* [ ] Unit tests have 80% coverage or greater, and are based on [officially supported technologies](https://folio-org.atlassian.net/wiki/spaces/TC/pages/5053681/Officially+Supported+Technologies)[^1] (3, 4)
  - Test files are present. Overall coverage of at least 80% and alignment of the testing technologies with the supported list have not been established by the evidence reviewed.
* [ ] Assigned to exactly one application descriptor within the FOLIO Community LSP Platform, specified in the Jira task for this module evaluation (3, 5)
  * _The FOLIO Community LSP Platform is defined at https://github.com/folio-org/platform-lsp._
  * _This can be evaluated by searching application descriptors across the folio-org GitHub organization (considering those applications part of the community platform) and confirming that this module only appears in the application descriptor of the one specified application._
  - Jira TCR-88 identifies `app-platform-minimal`, and the reviewed descriptor includes `mod-users`. A search confirming that the module appears in exactly one applicable Community LSP application descriptor has not been completed.
* [ ] S010 — Gracefully handles the absence of third party systems or related configuration.
  * _Note: This applies to optional third-party integrations and their configurations only. Required environment variables (those without sensible defaults) should fail fast on startup per the Environment Variables Policy._
The review examined HTTP integration, database access, profile-picture storage, and Kafka event-processing code. Some inspected paths propagate errors, but the review did not establish whether optional services and missing configurations are handled appropriately in all relevant cases. The profile-picture retrieval path can return 404 for caught storage exceptions; whether this response is appropriate for different failure conditions remains to be verified. Further targeted review is needed before a definitive conclusion can be drawn.

### Backend

Note: Backend criteria apply to modules, shared backend libraries, and edge modules.

* [ ] Module’s repository includes a compliant Module Descriptor (3, 5)
  * -_note: read more at https://github.com/folio-org/okapi/blob/master/okapi-core/src/main/raml/ModuleDescriptor.json_
  - The generated descriptor passed schema validation. The committed descriptor and all of its fields have not yet been checked against this criterion; the build result above is not sufficient by itself to mark this item acceptable.
* [ ] For each consumed API the module descriptor MUST include the interface requirement in the `"requires"` or `"optional"` section (3, 5)
  * _This is not applicable to libraries_
  - The [module descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/ModuleDescriptor-template.json) lists required and optional interfaces. A complete comparison against every consumed API has not been documented.
* [ ] Module includes executable implementations of all endpoints in the provides section of the Module Descriptor
  - The [RAML documentation](https://github.com/folio-org/mod-users/tree/v19.6.2/ramls), [module descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/ModuleDescriptor-template.json), and [UsersAPI.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/rest/impl/UsersAPI.java) were reviewed, but a complete comparison of every descriptor-provided endpoint against an executable implementation has not been completed.
* [ ] Environment variables comply with [FOLIO Policy](https://folio-org.atlassian.net/wiki/x/NYCgTg) (5, 11)
  * [ ] [All environment variables documented in ModuleDescriptor with required fields](https://folio-org.atlassian.net/wiki/spaces/TC/pages/1319141429/Environment+Variables#3.3-ModuleDescriptor-Enhancement)
  * [ ] [Variables properly classified as required or optional](https://folio-org.atlassian.net/wiki/spaces/TC/pages/1319141429/Environment+Variables#4.-Required-vs.-Optional-Variables)
  * [ ] [Variable names follow standardized naming conventions](https://folio-org.atlassian.net/wiki/spaces/TC/pages/1319141429/Environment+Variables#2.-Naming-Conventions)
  * [ ] [Module implements proper startup behavior](https://folio-org.atlassian.net/wiki/spaces/TC/pages/1319141429/Environment+Variables#4.2-Startup-Behavior)
  * [ ] [No sensitive information logged during startup configuration logging](https://folio-org.atlassian.net/wiki/spaces/TC/pages/1319141429/Environment+Variables#6.1-Startup-Configuration-Logging)
  - The [module descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/ModuleDescriptor-template.json) lists database, Kafka, and object-storage-related settings. The [README](https://github.com/folio-org/mod-users/blob/v19.6.2/README.md) and source code should be checked together because some configuration is read from environment variables or system properties. Please verify that all variables are documented and correctly classified, names follow the policy, startup behavior is appropriate, and sensitive values are not logged. Each subcriterion remains open pending that check.
* [ ] If a module provides interfaces intended to be consumed by other FOLIO Modules, they must be defined in the Module Descriptor "provides" section, and must conform to FOLIO [interface naming conventions](https://dev.folio.org/guidelines/naming-conventions/#interfaces) (3, 5)
  - The [module descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/ModuleDescriptor-template.json) includes provided interfaces. A complete check of names and versions against the conventions has not been documented.
* [ ] All API endpoints are documented in OpenAPI (11)
  * For existing module evaluation: RMB-based modules may continue using RAML instead.
  - [RAML documentation](https://github.com/folio-org/mod-users/tree/v19.6.2/ramls) was found for users, departments, groups, patron PIN, proxies, settings, staging users, and user-tenants. The shared [folio-custom-fields repository](https://github.com/folio-org/folio-custom-fields) contains documentation for the custom-fields endpoints. Because this is an existing RMB-based module, RAML is permitted by the template. The reviewer also confirmed official documentation for the tenant API: [tenant.raml](https://github.com/folio-org/raml/blob/raml1.0/ramls/tenant.raml), [tenant API reference](https://s3.amazonaws.com/foliodocs/api/raml/r/tenant.html), and the [Okapi tenant guide](https://github.com/folio-org/okapi/blob/master/doc/guide.md#enabling-for-a-tenant). A complete endpoint-to-documentation comparison remains to be completed.
* [ ] All API endpoints protected with appropriate permissions as per the [Permission Naming Guidelines](https://folio-org.atlassian.net/wiki/x/BgDtY), e.g. avoid using *.all permissions, all necessary module permissions are assigned, etc. (6)
  - Review whether each API endpoint is protected by an appropriate permission and whether permission names follow the [FOLIO Permission Naming Guidelines](https://folio-org.atlassian.net/wiki/x/BgDtY). The [module descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/ModuleDescriptor-template.json) declares permissions for the inspected handlers. However, a complete comparison of API endpoints against their required permissions has not been documented. This review therefore does not establish that all endpoints have appropriate permission declarations. Enforcement by Okapi, folio-kong, or the module sidecar is outside the scope of this criterion.
* [ ] Module provides reference data (if applicable), e.g. if there is a controlled vocabulary where the module requires at least one value (3, 16)
  - Address types and user groups were identified in the [reference-data resources](https://github.com/folio-org/mod-users/tree/v19.6.2/src/main/resources). The repository includes [ReferenceAndSampleDataMigrationIT.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/test/java/org/folio/rest/impl/ReferenceAndSampleDataMigrationIT.java), an automated integration test for reference and sample data migration. The [TenantRefAPI.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/rest/impl/TenantRefAPI.java) code also uses version-aware loading logic. This existing test should be considered as evidence when assessing reference-data loading and migration; any remaining questions should be limited to specific behavior not established by the test, including preservation of tenant-customized values where applicable.
* [ ] If provided, integration (API) tests must be written in an [officially supported technology](https://folio-org.atlassian.net/wiki/spaces/TC/pages/5053681/Officially+Supported+Technologies)[^1] (3, 4)
  * -_note: while it's strongly recommended that modules implement integration tests, it's not a requirement_
  * -_note: these tests are defined in https://github.com/folio-org/folio-integration-tests_
  - The Java `*IT.java` tests in [mod-users/src/test/java/org/folio/moduserstest](https://github.com/folio-org/mod-users/tree/v19.6.2/src/test/java/org/folio/moduserstest) are Maven verification tests; they are not the Karate API integration tests addressed by this criterion. This criterion refers to tests in the [FOLIO integration-tests repository for mod-users](https://github.com/folio-org/folio-integration-tests/tree/master/mod-users/src/test/java/org/folio). Review those Karate tests, if provided, to determine whether they use an officially supported technology. Integration tests are strongly recommended but are not required by this criterion.
* [ ] Data is segregated by tenant at the storage layer (6, 7)
  - The [RecordRepositoryImpl.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/service/impl/RecordRepositoryImpl.java) code uses tenant-specific PostgreSQL client selection, and [UserOutboxService.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/event/service/UserOutboxService.java) obtains the tenant ID from request headers. A complete isolation test using two tenants was not identified. This is a verification item, not a confirmed tenant-isolation defect.
* [ ] The module doesn’t access data in DB schemas other than its own and public. Exception: [FOLIO Query Machine](https://folio-org.atlassian.net/wiki/spaces/TC/pages/852852742/0011-Folio+Query+Machine+FQM) and its modules are the mechanism through which we provide read access across DB storage boundaries. (6, 7)
  - A limited search of the [database schema resources](https://github.com/folio-org/mod-users/tree/v19.6.2/src/main/resources) found `public.uuid_generate_v4()` in a configuration-table SQL script and did not identify cross-schema data access in the reviewed [RecordRepositoryImpl.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/service/impl/RecordRepositoryImpl.java). This limited check does not establish module-wide compliance.
* [ ] Any dependencies, other than on defined interfaces, are declared in the README.md.
  - The [module descriptor template](https://github.com/folio-org/mod-users/blob/v19.6.2/descriptors/ModuleDescriptor-template.json) lists optional dependencies on `feesfines` and `configuration`. Verify that the [README](https://github.com/folio-org/mod-users/blob/v19.6.2/README.md) explains dependencies beyond defined interfaces, including their purpose.
* [ ] The module responds with a tenant’s content based on x-okapi-tenant header (7)
  - The [AbstractRestTest.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/test/java/org/folio/moduserstest/AbstractRestTest.java) test setup sets the tenant header, and [RecordRepositoryImpl.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/service/impl/RecordRepositoryImpl.java) uses tenant-aware database access. Response isolation and consistent header handling across endpoints have not been verified.
* [ ] Standard GET /admin/health endpoint returning a 200 response (5)
  * -_note: read more at https://folio-org.atlassian.net/wiki/spaces/DD/pages/1777917/Back+End+Module+Health+Check+Protocol_
  - [InstallUpgradeIT.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/test/java/org/folio/rest/InstallUpgradeIT.java#L43) creates a Docker image, deploys it in the build environment, and tests `GET /admin/health`, expecting HTTP 200 with body `OK`. The reviewer provided a [successful automated build run](https://github.com/folio-org/mod-users/actions/runs/38407868759/job/115332182136) that executes `InstallUpgradeIT`. This confirms that the health endpoint is covered by a test that runs in the automated build pipeline.
* [ ] High Availability (HA) compliant (5, 14, 15)
  * Possible red flags:
    * Connection affinity / sticky sessions / etc. are used
    * Local container storage is used
    * Services are stateful
  - The [ProfilePictureStorage.java](https://github.com/folio-org/mod-users/blob/v19.6.2/src/main/java/org/folio/service/storage/ProfilePictureStorage.java) profile-picture storage path uses an object-storage client with tenant-specific keys. That does not establish module-wide high-availability compliance. Verify that the module does not rely on local state, sticky sessions, or single-instance behavior.
* [ ] Module only uses infrastructure / platform technologies on the [officially supported technologies](https://folio-org.atlassian.net/wiki/spaces/TC/pages/5053681/Officially+Supported+Technologies) list.[^1]
  * _e.g. PostgreSQL, ElasticSearch, etc._ (3, 5, 12)
  - The [Maven configuration](https://github.com/folio-org/mod-users/blob/v19.6.2/pom.xml) and inspected source use Java 21, Vert.x, Kafka, PostgreSQL, and FOLIO object-storage components. Verify all relevant direct and transitive technologies against the supported list for the release being evaluated; the automated evaluator's source revision has not yet been reconciled with `v19.6.2`.

## TCR Process Improvements

[*Please include here any suggestions that you feel might improve the TCR Processes.*]

##

[^1]: Refer to the [Officially Supported Technologies](https://folio-org.atlassian.net/wiki/spaces/TC/pages/5053681/Officially+Supported+Technologies) page for the most recent ACTIVE release.

