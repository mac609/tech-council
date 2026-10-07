# mod-users

**Jira:** TCR-88
**Evaluation date:** 2026-10-06
**Application:** app-platform-minimal
**Module type:** Backend

**Source under evaluation:**
TO BE CONFIRMED. Current candidate: [mod-users v19.6.2](https://github.com/folio-org/mod-users/releases/tag/v19.6.2), commit `361c1ff`.

**Module description:**
Stores and serves the central user record (identity, contact info, addresses, status, profile picture) for a FOLIO tenant. Other modules call this API to look up or modify the canonical user data; authentication, credentials, and permissions live in separate modules (mod-login-keycloak, mod-permissions, etc.).

**Related modules:**

* `mod-users-bl` — business-logic/join layer providing composite user information.
* `mod-users-keycloak` — orchestrates user operations between FOLIO `mod-users` and Keycloak.
* `mod-permissions` — maintains permission/user associations related to FOLIO users.
* `ui-users` — FOLIO UI for searching, viewing, creating, and editing users.
* `app-platform-minimal` — application descriptor containing `mod-users`.
* `mod-settings` — used by current `mod-users` functionality for user settings/configuration.

**Points of contact:**
TO BE DETERMINED.

**Self-evaluation:**
TO BE DETERMINED.

---

# Values and Criteria Evaluation

## Administrative

### Listed by the Product Council with a positive evaluation result

**Status:** INAPPLICABLE

**Comment:**
This is an existing-module evaluation. The acceptance criteria specify that this criterion is inapplicable for existing module evaluation.

---

# Shared/Common

### Uses Apache 2.0 license

**Status:** ACCEPTABLE

**Evidence:**
The `mod-users` repository identifies the software as being distributed under the Apache License, Version 2.0 and includes a LICENSE file.

**Comment:**
The repository states that `mod-users` is licensed under Apache 2.0.

---

### Module build must produce a valid module descriptor

**Status:** TO BE DETERMINED

**Evidence to collect:**

* `descriptors/` contents
* generated `target/ModuleDescriptor.json`
* successful Maven build
* module descriptor validation

**Comment:**
The repository documents the module descriptor and identifies `target/ModuleDescriptor.json` as the built descriptor. The actual descriptor should be generated and inspected for this evaluation.

---

### Third-party dependencies comply with ASF 3rd Party License Policy

**Status:** TO BE DETERMINED

**Evidence to collect:**

* `pom.xml`
* dependency tree
* FOLIO Module Evaluator results
* identification of any LGPL or other restricted dependencies

**Comment:**
The acceptance criteria specifically identify third-party dependency licensing as an evaluation requirement. This requires an actual dependency evaluation rather than inference from the repository license.

---

### Installation documentation is included

**Status:** TO BE DETERMINED

**Evidence to collect:**

* README installation/configuration instructions
* Module Descriptor
* environment-variable documentation

**Comment:**
The repository contains a README with configuration information, including profile-picture configuration. The completeness of installation documentation needs to be evaluated.

---

### Personal data form is completed and accurate

**Status:** TO BE DETERMINED

**Evidence to collect:**

* `PERSONAL_DATA_DISCLOSURE.md`

**Comment:**
Confirm that the file exists, is complete, and accurately describes the personal data handled by `mod-users`.

---

### Sensitive and environment-specific information is not checked into git

**Status:** TO BE DETERMINED

**Evidence to collect:**

* repository search for credentials/secrets
* configuration files
* example configuration
* source code
* Git history where appropriate

**Comment:**
This criterion should receive particular attention because `mod-users` handles personal user data and includes profile-picture functionality.

---

### No default passwords, access keys, private keys, or other credentials

**Status:** TO BE DETERMINED

**Evidence to collect:**

* repository-wide secret scan
* configuration files
* tests
* environment-variable documentation

**Comment:**
Profile-picture configuration includes an encryption-key setting. Verify that the repository does not provide an operational default credential or secret.

---

### Uses officially supported language/framework

**Status:** TO BE DETERMINED

**Evidence:**
The repository currently identifies Java 21 as a prerequisite and uses the raml-module-builder framework.

**Comment:**
The official supported-technologies list should be checked to confirm that the Java/framework combination used by the evaluated version is supported.

---

### Uses FOLIO interfaces provided by previously accepted modules

**Status:** TO BE DETERMINED

**Evidence to collect:**

* `descriptors/ModuleDescriptor.json`
* required and optional interfaces
* related module evaluations

**Comment:**
The module descriptor needs to be compared with the accepted FOLIO interfaces available to the application.

---

### Does not depend on an unapproved FOLIO library

**Status:** TO BE DETERMINED

**Evidence to collect:**

* `pom.xml`
* dependency tree
* FOLIO Module Evaluator results

---

### Gracefully handles absence of optional third-party systems/configuration

**Status:** TO BE DETERMINED

**Evidence to collect:**

* Module Descriptor environment configuration
* README
* startup behavior
* optional profile-picture/object-storage configuration
* tests

**Comment:**
The profile-picture functionality can use database storage or object storage. Verify that optional infrastructure is handled according to FOLIO policy.

---

### SonarQube has no security/high-severity issues or excessive duplication

**Status:** TO BE DETERMINED

**Evidence to collect:**

* SonarQube report for the evaluated version/commit
* security issues
* reliability issues
* duplication percentage
* suppressed/disabled rules

**Comment:**
This cannot be established from the repository README alone.

---

### Uses officially supported build tools

**Status:** TO BE DETERMINED

**Evidence:**
The repository currently identifies Maven as the build tool and Java 21 as a prerequisite.

**Comment:**
Confirm against the current officially supported technologies list.

---

### Unit tests have at least 80% coverage

**Status:** TO BE DETERMINED

**Evidence to collect:**

* Maven test results
* JaCoCo/coverage report or equivalent
* CI results

**Comment:**
The acceptance criterion requires at least 80% unit-test coverage.

---

### Assigned to exactly one application descriptor

**Status:** TO BE DETERMINED

**Evidence:**
TCR-88 identifies `app-platform-minimal` as the application.

**Comment:**
The acceptance criteria require searching the FOLIO GitHub organization to determine whether the module appears in exactly one applicable Community LSP application descriptor. The current `app-platform-minimal` template includes `mod-users`. A broader organization-wide search is still required.

---

# Backend

### Repository includes a compliant Module Descriptor

**Status:** TO BE DETERMINED

**Evidence to collect:**

* `descriptors/`
* generated `ModuleDescriptor.json`

---

### Every consumed API is declared in `requires` or `optional`

**Status:** TO BE DETERMINED

**Evidence to collect:**
Compare actual API/interface usage in source code with the Module Descriptor.

---

### All endpoints in the Module Descriptor have executable implementations

**Status:** TO BE DETERMINED

**Evidence to collect:**
Compare `provides` interfaces/endpoints in the Module Descriptor against the implementation and RAML definitions.

---

### Environment variables comply with FOLIO policy

**Status:** TO BE DETERMINED

**Evidence to collect:**

* Module Descriptor environment-variable definitions
* README
* startup configuration
* source code

Verify required/optional classification, naming, startup behavior, and sensitive-value logging.

---

### Provided interfaces are correctly declared and named

**Status:** TO BE DETERMINED

**Evidence to collect:**
Module Descriptor `provides` section and FOLIO interface naming conventions.

---

### All API endpoints are documented

**Status:** ACCEPTABLE / TO BE CONFIRMED

**Evidence:**
The repository contains RAML API definitions, including `ramls/users.raml`. The current acceptance criteria specifically allow existing RMB-based modules to continue using RAML rather than OpenAPI.

**Comment:**
The repository documents the Users API using RAML. Final verification should compare the RAML definitions with the actual Module Descriptor endpoints.

---

### API endpoints are protected with appropriate permissions

**Status:** TO BE DETERMINED

**Evidence to collect:**

* permission definitions
* RAML
* Module Descriptor
* source code
* tests

**Comment:**
This is a particularly important criterion for `mod-users`, because the module manages canonical user records. Verify that read, create, update, delete, configuration, and other sensitive operations have appropriately scoped permissions and do not rely unnecessarily on broad `*.all` permissions.

---

### Reference data is provided where applicable

**Status:** TO BE DETERMINED / POSSIBLY INAPPLICABLE

**Comment:**
Determine whether `mod-users` has any required controlled vocabulary/reference data that must be initialized by the module.

---

### Integration/API tests use supported technology

**Status:** TO BE DETERMINED

**Comment:**
Integration tests are recommended but not mandatory. If provided, verify that they use an officially supported technology.

---

### Data is segregated by tenant at the storage layer

**Status:** TO BE DETERMINED

**Evidence to collect:**

* database schema definitions
* RMB configuration
* tenant initialization
* integration tests
* multi-tenant testing

**Comment:**
This is a critical criterion for `mod-users`. Verify that user data is physically/logically isolated by tenant at the storage layer.

---

### Module does not access another module's private database schema

**Status:** TO BE DETERMINED

**Evidence to collect:**

* SQL/database access code
* schema definitions
* database scripts
* source-code search for cross-module schemas

**Comment:**
The module should access its own database schema and public/shared schemas only, except where FOLIO Query Machine rules apply.

---

### Dependencies other than defined interfaces are documented

**Status:** TO BE DETERMINED

**Evidence to collect:**

* README
* Module Descriptor
* source code
* deployment documentation

---

### Responds with tenant content based on `x-okapi-tenant`

**Status:** TO BE DETERMINED

**Evidence to collect:**

* request handling code
* tenant initialization
* database access
* multi-tenant integration tests

**Comment:**
Verify both normal reads/writes and error conditions.

---

### Standard `GET /admin/health` returns HTTP 200

**Status:** TO BE DETERMINED

**Evidence to collect:**

* endpoint implementation
* Module Descriptor
* integration test
* running module test

---

### High Availability compliant

**Status:** TO BE DETERMINED

**Evidence to collect:**

* application state
* database access
* local filesystem usage
* caching
* session/state management
* deployment configuration

**Comment:**
Check specifically for sticky-session requirements, local persistent state, stateful services, and local container storage.

---

### Uses only officially supported infrastructure/platform technologies

**Status:** TO BE DETERMINED

**Evidence to collect:**

* PostgreSQL/database usage
* Kafka usage
* S3/object storage
* other external infrastructure
* supported-technologies list

---

# Preliminary observations

The following relationships should be considered during the evaluation:

* `mod-users` is the central user-data module.
* `mod-users-bl` is a business-logic/join module that provides composite user information and describes itself as a front end for `mod-users` and `mod-permissions`.
* `mod-users-keycloak` coordinates user operations between `mod-users` and Keycloak and specifically warns that direct modification of certain users through `mod-users` can create inconsistencies with Keycloak.
* `mod-permissions` maintains its own permission/user associations and refers to the FOLIO user ID from `mod-users`.
* `app-platform-minimal` currently includes `mod-users` as one of its backend modules.

These relationships should be documented rather than treating `mod-users` as an isolated component.

# Evaluation status

This is a **working evaluation draft**. Criteria marked `TO BE DETERMINED` require additional evidence before an ACCEPTABLE or UNACCEPTABLE status can be assigned.

The evaluation should not be considered complete until:

1. TCR-88 identifies/accepts the exact source tag or commit.
2. The self-evaluation is available.
3. The Module Descriptor is inspected.
4. Dependency/license analysis is performed.
5. Test coverage is verified.
6. SonarQube results are verified.
7. Permissions are reviewed.
8. Tenant isolation is verified.
9. Application-descriptor membership is verified.
10. Health and HA behavior are verified.

