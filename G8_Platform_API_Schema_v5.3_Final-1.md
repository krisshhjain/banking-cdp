# Group 8 Platform — API Schema Specification

> Aligned to Platform Data Model v5.3 | MIRAI BFSI Digital Twin | Build the Twin

> **Source:** G8_Platform_API_Schema_v5.3_Final-1.pdf, 92 pages. **Conversion:** extracted PDF text and tables with source diagrams retained as local images. This is a transcription of the supplied working specification, not an approved implementation contract.

> **Note:** The source itself identifies owner decisions and proposed routes that require approval. Check the original PDF for exact visual layout and any overlapped text.

---

<!-- PDF page 1: Cover -->

GROUP 8 PLATFORM
API SCHEMA SPECIFICATION


Aligned to Platform Data Model


v5.3


MIRAI BFSI Digital Twin  |  Build the Twin


Document ID
G8_PLATFORM_API_SCHEMA_V5.3_FINAL


Version
7.2 - aligned against Data Model v5.3


Prepared by
Group 8 Platform API Team


Date
11 November 2026


36 active APIs; former transaction-limit API 18 removed; cards renumbered 01-36


Scope


Status
Working specification - proposed routes and owner decisions require approval


Primary source
Group8 Platform Data Model v5.3 Flow + Sample Data v5.3


Wireframe reference
Group8 Platform Wireframes v2 (incomplete; reference only)


<!-- PDF page 2 -->

## Document Control and Interpretation


This revision preserves the detailed API-card contract style, all remaining request/success/error examples and End-to-End API flowcharts.
The former transaction-limit API 18 is removed; the remaining 36 cards are renumbered consecutively from API 01 through API 36.


Data Model v5.3 is the source of truth for Platform-owned tables, column names, types, keys, relationships and enumerations. It
contains 18 tables: 10 master and 8 transactional, arranged in seven steps. The data model defines database structure, not complete
HTTP endpoint paths or payload contracts.


### DATA MODEL v5.3 SEQUENCE


| 1 | Groups, services, menus domain_group -&gt; service_registry -&gt; menu_item MASTER |
| --- | --- |
| 2 | Personas and permissions persona -&gt; permission -&gt; persona_permission MASTER |
| 3 | APIs, settings, bank date route -&gt; config_setting -&gt; bank_clock -&gt; business_calendar MASTER |
| 4 | Actors and their personas actor -&gt; actor_persona -&gt; admin_profile TRANSACTIONAL |
| 5 | Session session -&gt; session_sync_event TRANSACTIONAL |
| 6 | Every request idempotency_record -&gt; audit_event TRANSACTIONAL |
| 7 | Admin workflow: changes change_request TRANSACTIONAL |


KEY RULES APPLIED TO EVERY API CARD
- Use only the 18 v5.3 tables and their exact columns, types, keys and enum values. Payment limits and rail cut-offs are Group 4-owned.
- Group 1 user_id, party_id, employee_id and auth_session_ref/login_ref remain plain external identifiers where specified; do not invent Platform
foreign keys.
- UUID fields remain UUIDs. String identifiers such as group_id (G1?G8), clock_id (MAIN), event_id, idempotency_key, login_ref, user_id, party_id and
employee_id retain their source formats.
- actor_persona.status = PENDING_APPROVAL/ACTIVE/REVOKED; session.status = ACTIVE/LOGGED_OUT/TIMED_OUT/REVOKED.
- Success/error envelopes remain data/meta and error/meta. List responses retain page.next_cursor/page.limit.
- Gateway derives actor/session identity from the authenticated caller. Never trust browser-supplied identity for /me or /current operations.
- Do not store raw passwords, OTPs or raw login tokens. session.login_response must be redacted.


<!-- PDF page 3 -->

## Data Model v5.3 Review - Corrections and Clarifications


The v5.3 PDF and Sample Data workbook define 18 Platform tables: 10 master and 8 transactional, arranged across seven steps. The former API
material included payment-limit functionality that is not part of the v5.3 Group 8 inventory. The following are documentation and constraint
clarifications; they do not redesign the approved data model.


### REQUIRED ALIGNMENT CORRECTIONS - Remove the former payment-limit table from Group 8 inventory, API source mappings and Appendix A. Payment limits and rail cut-offs are Group


4-owned; the former transaction-limit API 18 is removed and remaining cards are renumbered consecutively.

- Step 7 is the approved master-data change workflow. Ordinary master-data edits must submit change_request and must not directly mutate
master data. Any demo exception needs explicit owner approval.
- business_calendar contains calendar_date, is_working_day, holiday_name, updated_at and updated_by. Do not add NEFT/RTGS cut-off fields
to Group 8 responses.
- session uses the three-column FK (actor_persona_id, actor_id, persona_id) to actor_persona. Treat it as one composite relationship, not three
independent references.
- persona_permission uses composite PK (persona_id, permission_id); idempotency_record uses composite PK (idempotency_key, actor_id);
session_sync_event uses composite PK (event_id, direction).
- bank_clock is intended to have only clock_id=MAIN. A primary key alone does not enforce a single row; enforce this through a database
constraint or controlled-write rule.


OWNER DECISIONS TO CONFIRM
1. session_sync_event: workbook description calls event_id unique, while the PK is (event_id, direction). Confirm global uniqueness versus
uniqueness within each direction.
2. idempotency_record: description says money-call protection, while a sample fingerprint describes a policy nominee change. Confirm whether all
mutations or only money operations are in scope.
3. audit_event.outcome: allowed values are CONFIRMED, FAILED, PENDING and UNKNOWN. Define how successful reads and non-payment
actions are represented; do not add enum values without approval.
4. Step 7 target scope: change_request.target_table includes ACTOR_PERSONA and ADMIN_PROFILE although the diagram says approved
changes update master data in Steps 1?4. Confirm whether these transactional targets are intentionally supported.
5. PERSONA_PERMISSION has a composite key but change_request.target_id is varchar. Approve a canonical composite-key encoding.
6. Confirm duplicate active actor_persona assignment rules, configuration effective-version overlap/precedence, bank timezone and bank_clock
singleton enforcement.
7. actor_persona has no REJECTED status. Decide whether a rejected request remains represented by another status or whether the model itself
is amended.


These points require Data Model/programme-owner decisions before implementation. The API specification must not invent columns, constraints
or enum values to resolve them.


<!-- PDF page 4 -->

## Wireframe-to-API Mapping (Reference Only)


Wireframes v2 defines Group 8 Home, All Services and Status pages plus the shared header/menu/footer. It does not define domain screens
owned by Groups 1-7. Admin screens remain proposals derived from the Data Model and require UI confirmation.


Home before login
Group 1 owns login. No Group 8 login form or Platform customer-data read.


Home after login / shared header
Group 1 supplies authentication/profile dependency; Platform validates its current session. Logout ends the current Platform session and
follows the approved Group 1 sync contract.


Menu and All Services
GET /registry/v1/me/services. Derive access through session -> actor_persona -> persona_permission -> permission -> menu_item ->
service_registry. Do not introduce user_service_access; it is not in v5.3.


Status
GET /registry/v1/status. Show service_registry status separately from the external health_url monitoring result. Health is not a persisted
service_registry column.


Admin - Groups / Services / Menus
Read domain_group, service_registry and menu_item. Proposed master-data changes must follow Step 7 maker-checker; no direct ordinary
master-table write.


Admin - Personas / Permissions
Read persona, permission and persona_permission. Proposed master-data changes must follow Step 7. persona_permission uses the
composite key (persona_id, permission_id).


Admin - Routes / Settings / Clock / Calendar
Read route, config_setting, bank_clock and business_calendar. Payment limits and rail cut-offs are Group 4-owned, not Group 8 tables.


Admin - People / Sessions / Audit / Changes
Use actor, actor_persona, admin_profile, session, session_sync_event, idempotency_record, audit_event and change_request. Step 7 target
scope for selected transactional tables requires owner confirmation.


Ownership boundary: Group 1 owns authentication and external identity identifiers. Group 8 owns Platform registry, RBAC, actors, Platform
sessions, Gateway, settings, bank clock, business calendar and change workflow. Group 4 owns payment limits and rail cut-offs.


<!-- PDF page 5 -->

## Common API Contract Standard


| Topic |  | Contract standard |
| --- | --- | --- |
| URL pattern |  | /{service}/v1/{resource}. Proposed route names remain subject to owner approval. |
| Identity |  | Resolve actor_id/session from authenticated login pass. party_id/user_id/employee_id/login_ref are Group 1 values and are not browser-selected identifiers. |
| Headers |  | Customer: Authorization: Bearer &lt;login-pass&gt;, Accept, X-Request-ID; JSON mutations also Content-Type. Staff uses staff login pass and Gateway-validated permissions. SERVICE calls use trusted service identity. Session start and Group 1 sync are service-to-service. |
| Methods |  | GET reads; POST creates/commands; PATCH partial update; DELETE terminates/deletes only where lifecycle allows. The route.method enum in is GET, POST, PATCH, DELETE. v5.3 |
| Success envelope |  | {"data": {...}, "meta": {"request_id":"...","trace_id":"...","timestamp":"...Z","simulated":true, "source":"..."} } |
| List envelope |  | {"data":[...],"page":{"next_cursor":null,"limit":20},"meta": {"request_id":"...","trace_id":"...","timestamp":"...Z","simulated":true} } |
| Error envelope |  | {"error":{"code":"VALIDATION_FAILED","message":"Customer-safe message","details": [{"field":"...","issue":"..."}],"retryable":false},"meta": {"request_id":"...","trace_id":"...","timestamp":"...Z","simulated":true} } |
| HTTP behaviour |  | 400 validation; 401 unauthenticated; 403 forbidden/same-person; 404 not found; 409 conflict; 429 rate limited with Retry-After; 503 unavailable; 504 upstream timeout/outcome unknown. |
| Pagination |  | List endpoints accept limit/cursor and return page.next_cursor/page.limit. Empty list is HTTP 200 with data: [] and next_cursor: null. |
|  | Money/date | Group 8 v5.3 tables do not define payment-limit amount fields. Any payment amount or limit contract belongs to its owning group. Dates use ISO YYYY-MM-DD; timestamps use ISO 8601 UTC. |
| Retries |  | Use Idempotency-Key for protected mutations. Never blindly retry payment or any operation with uncertain outcome; query status first. |


v5.3 ALIGNMENT NOTES


Idempotency is keyed by (idempotency_key, actor_id), not globally by key. The workbook describes money-call protection but includes a
non-payment mutation example; the supported scope remains an owner decision. The Data Model contains no Group 8 transaction-limit or
rail cut-off fields.


<!-- PDF page 6 -->

## API Card Navigation Index


| Data Model Step | API Card |  | Operation | Endpoint / operation |
| --- | --- | --- | --- | --- |
| 1 | API CARD 01 |  | List registered domain groups | GET /registry/v1/groups |
|  | API CARD 02 |  | List all registered services | GET /registry/v1/services |
|  | API CARD 03 |  | Register a service | POST /registry/v1/services |
|  | API CARD 04 |  | Update a registered service | PATCH /registry/v1/services/{service_id} |
|  | API CARD 05 |  | List services available to the signed-in user | GET /registry/v1/me/services |
|  | API CARD 06 |  | Read service status | GET /registry/v1/status |
|  | API CARD 07 |  | List menu items for a service | GET /registry/v1/menu-items? service_id={service_id} |
|  | API CARD 08 |  | Create a menu item | POST /registry/v1/menu-items |
|  | API CARD 09 |  | Update a menu item | PATCH /registry/v1/menu-items/{menu_item_id} |
| 2 | API CARD 10 |  | List personas | GET /access/v1/personas |
|  | API CARD 11 |  | Create a persona | POST /access/v1/personas |
|  | API CARD 12 |  | List permissions | GET /access/v1/permissions |
|  | API CARD 13 |  | Create a permission | POST /access/v1/permissions |
|  | API CARD 14 |  | Read permissions assigned to a persona | GET /access/v1/personas/{persona_id}/permissions |
|  | API CARD 15 |  | Assign a permission to a persona | PUT /access/v1/personas/{persona_id}/permissions/{permission_id} |
| 3 | API CARD 16 |  | List configured API routes | GET /gateway/v1/routes |
|  | API CARD 17 |  | Read a configuration setting | GET /internal/v1/settings/{config_key} |
|  | API CARD 18 | API CARD 18 | Read simulated bank date and time | GET /clock/v1/today |
|  | API CARD 19 |  | Read business calendar | GET /clock/v1/calendar |
|  | API CARD 20 |  | Advance simulated bank date/time | POST /clock/v1/advance |
|  | API CARD 21 |  | Check service health | GET /health |
| 4 | API CARD 22 |  | List Platform actors | GET /access/v1/actors |
|  | API CARD 23 |  | List personas assigned to an actor | GET /access/v1/actors/{actor_id}/personas |
|  | API CARD 24 |  | Request a persona assignment | POST /access/v1/actors/{actor_id}/personas |
|  | API CARD 25 |  | Approve or reject a persona assignment | POST /access/v1/actor-persona-assignments/{actor_persona_id}/ |


<!-- PDF page 7 -->

decision


API CARD 26


Read the current admin profile
GET 
/access/v1/admin-profile/me


API CARD 27


POST /sessions/v1/sessions


Start a Platform session after 
Group 1 confirms login


API CARD 28


Check the current Platform 
session


GET 
/sessions/v1/sessions/current


API CARD 29


End the current Platform session 
(logout)


DELETE 
/sessions/v1/sessions/current


5


API CARD 30


Receive or retry a Group 1 
session-sync event


POST /internal/v1/session-sync-
events


API CARD 31


End all sessions after Group 1 
revokes a customer login


POST 
/sessions/v1/parties/{party_id}/se
ssions/end-all


API CARD 32


Gateway idempotency 
processing (internal contract)


Gateway request interception 
(no customer-facing endpoint)


6


API CARD 33


Search Gateway audit events
GET /audit/v1/events


API CARD 34


Submit an administrative change 
request


POST /internal/v1/change-
requests


API CARD 35


Read a change request
GET /internal/v1/change-
requests/{change_id}


7


API CARD 36


Approve or reject a change 
request


POST /internal/v1/change-
requests/{change_id}/decision


The former API 18 (transaction-limit functionality) has been removed because it is Group 4-owned and absent from the v5.3 Group 8 model. The remaining 36
active cards are renumbered consecutively.


<!-- PDF page 8 -->

## STEP 1 | API CARD 01 — List registered domain groups


**Data Model table(s): domain_group**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Platform Admin opens the Groups registry. |
| Method + endpoint | GET /registry/v1/groups |
| Owner microservice | registry-service |
| Who can call | STAFF with registry-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Optional status filter; cursor and limit. Status must be ACTIVE or INACTIVE. |
| Source tables / dependencies | registry.domain_group |
| Backend flow | Read domain_group ordered by group_id; apply status filter and cursor. Do not expose created_by/updated_by unless the caller has audit permission. |
| Frontend response | Return group_id, group_name, description and status for the administration list. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether inactive groups are included by default and the maximum page size. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/registry/v1/groups",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "status": "ACTIVE",
      "limit": 20
    }
  }
}
```


### Success response example (JSON)


```json
{
  "data": [
    {
      "group_id": "G8",
      "group_name": "Platform",
      "description": "Gateway, sessions, RBAC, admin panel, registry, settings and bank date",
      "status": "ACTIVE"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 20
  },
  "meta": {
    "request_id": "req_G8_001",
    "trace_id": "trace_G8_001",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "registry-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 9 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_01",
    "trace_id": "trace_ERR_01",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 9](G8_Platform_API_Schema_assets/workflow_page_09.png)


<!-- PDF page 10 -->

## STEP 1 | API CARD 02 — List all registered services


**Data Model table(s): service_registry**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Staff opens the service registry administration page. |
| Method + endpoint | GET /registry/v1/services |
| Owner microservice | registry-service |
| Who can call | STAFF with service-registry read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Optional owner_group and status filters; cursor/limit. status PLANNED, ∈ IN_BUILD, LIVE, DISABLED, RETIRED. |
| Source tables / dependencies | registry.service_registry; registry.domain_group |
| Backend flow | Read service_registry and join domain_group for owner label. Use only fields in the model: service_id, owner_group, service_name, description, base_url, endpoint_path, health_url, contract_version, status. v5.3 |
| Frontend response | Populate the staff registry. Do not show legacy display_name, page_url, menu_label, readiness or customer_facing fields; they are not in . v5.3 |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether health state is included here or only in the separate Status endpoint. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/registry/v1/services",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "owner_group": "G8",
      "status": "LIVE",
      "limit": 20
    }
  }
}
```


### Success response example (JSON)


<!-- PDF page 11 -->

```json
{
  "data": [
    {
      "service_id": "d1000000-0000-4000-8000-000000000003",
      "owner_group": "G8",
      "service_name": "Session Service",
      "description": "Opens and ends Platform sessions",
      "base_url": "https://platform.mirai-bank.test",
      "endpoint_path": "/sessions",
      "health_url": "https://platform.mirai-bank.test/sessions/health",
      "contract_version": "1.0.0",
      "status": "LIVE"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 20
  },
  "meta": {
    "request_id": "req_G8_002",
    "trace_id": "trace_G8_002",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "registry-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_02",
    "trace_id": "trace_ERR_02",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


<!-- PDF page 12 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 12](G8_Platform_API_Schema_assets/workflow_page_12.png)


<!-- PDF page 13 -->

## STEP 1 | API CARD 03 — Register a service


**Data Model table(s): service_registry**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | Authorised administrator registers a microservice in the Platform. |  |
| Method + endpoint | POST /registry/v1/services |  |
| Owner microservice | registry-service |  |
| Who can call | STAFF with service-create permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Required: owner_group, service_name, description, base_url, endpoint_path, health_url, contract_version, status. service_id is generated by Platform unless a seed/import process supplies it. owner_group must exist in domain_group. status PLANNED, IN_BUILD, LIVE, DISABLED, ∈ RETIRED. Required: owner_group, owner_pod, service_name, description, base_url, endpoint_path, health_url, contract_version and status. service_id is generated by Platform. owner_pod is mandatory in v5.3. |  |
|  |  | endpoint_path, health_url, contract_version, status. service_id is generated by Platform unless a seed/import process supplies it. owner_group must exist in domain_group. status PLANNED, IN_BUILD, LIVE, DISABLED, ∈ RETIRED. Required: owner_group, owner_pod, service_name, description, base_url, endpoint_path, health_url, contract_version and status. service_id is generated by Platform. owner_pod is mandatory in v5.3. |
| Source tables / dependencies | registry.service_registry; registry.domain_group; access.actor (created_by) |  |
| Backend flow | Validate owner_group FK, URL formats, unique service_name and allowed status. Generate UUID service_id. Set created_at and created_by from authenticated actor; write audit_event. |  |
|  |  | Validate owner_group FK, URL formats, unique service_name and allowed status. Generate UUID service_id. Set created_at and created_by from authenticated actor; write audit_event. |
| Frontend response | Show the created registry row. Customer menu visibility is derived separately from actor/persona permissions and menu_item; do not infer visibility from service status alone. |  |
| Safe to repeat? | Idempotent with the same Idempotency-Key and identical request fingerprint. |  |
| Open points / owner decisions | Confirm whether service_name uniqueness is case-sensitive and whether status may be set directly to LIVE. |  |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/registry/v1/services",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "owner_group": "G8",
    "service_name": "Session Service",
    "description": "Opens and ends Platform sessions",
    "base_url": "https://platform.mirai-bank.test",
    "endpoint_path": "/sessions",
    "health_url": "https://platform.mirai-bank.test/sessions/health",
    "contract_version": "1.0.0",
    "status": "PLANNED"
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "service_id": "d1000000-0000-4000-8000-000000000005",
    "owner_group": "G8",
    "service_name": "Session Service",
    "description": "Opens and ends Platform sessions",
    "base_url": "https://platform.mirai-bank.test",
    "endpoint_path": "/sessions",
    "health_url": "https://platform.mirai-bank.test/sessions/health",
    "contract_version": "1.0.0",
    "status": "PLANNED",
    "created_at": "2026-11-10T12:00:00Z",
    "created_by": "a1000000-0000-4000-8000-000000000901"
  }
}
```


### Error response example (HTTP 409 Conflict)


<!-- PDF page 14 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "A service with the same unique name already exists.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_03",
    "trace_id": "trace_ERR_03",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 14](G8_Platform_API_Schema_assets/workflow_page_14.png)


<!-- PDF page 15 -->

## STEP 1 | API CARD 04 — Update a registered service


**Data Model table(s): service_registry**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | Staff edits an existing service’s description, base URL, endpoint path, health URL, contract version or status. |  |
| Method + endpoint | PATCH /registry/v1/services/{service_id} |  |
| Owner microservice | registry-service |  |
| Who can call | STAFF with service-update permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Path service_id is UUID. Body contains only permitted fields from service_registry; service_id, created_at and created_by are immutable. status must use the enum. v5.3 |  |
| Source tables / dependencies | registry.service_registry; access.actor; gateway.audit_event |  |
| Backend flow |  | Load target row; validate partial fields and status enum; update supplied fields and updated_at/updated_by; preserve omitted values. |
| Frontend response | Refresh the service registry and dependent navigation data after the update. |  |
| Safe to repeat? | PATCH is not assumed safe to replay unless Idempotency-Key and same body are reused. |  |
| Open points / owner decisions | Confirm mutable field allowlist, optimistic concurrency/version handling and whether status changes require change_request approval. |  |


### Request example


```json
{
  "method": "PATCH",
  "endpoint": "/registry/v1/services/{service_id}",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "service_id": "d1000000-0000-4000-8000-000000000003",
    "status": "LIVE",
    "contract_version": "1.0.1"
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "service_id": "d1000000-0000-4000-8000-000000000003",
    "owner_group": "G8",
    "service_name": "Session Service",
    "description": "Opens and ends Platform sessions",
    "base_url": "https://platform.mirai-bank.test",
    "endpoint_path": "/sessions",
    "health_url": "https://platform.mirai-bank.test/sessions/health",
    "contract_version": "1.0.1",
    "status": "LIVE",
    "updated_at": "2026-11-10T12:05:00Z",
    "updated_by": "a1000000-0000-4000-8000-000000000901"
  }
}
```


### Error response example (HTTP 404 Not Found)


<!-- PDF page 16 -->

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "The requested service does not exist.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_04",
    "trace_id": "trace_ERR_04",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 16](G8_Platform_API_Schema_assets/workflow_page_16.png)


<!-- PDF page 17 -->

## STEP 1 | API CARD 05 — List services available to the signed-in user


**Data Model table(s): service_registry + menu_item + persona_permission**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Customer opens Home after login, All Services, or shared navigation. |
| Method + endpoint | GET /registry/v1/me/services |
| Owner microservice | registry-service |
| Who can call | CUSTOMER; resolved actor/session required |
| Required headers | Authorization: Bearer &lt;login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | No party_id or actor_id in the request. Derive actor/persona from current session. Optional cursor/limit. |
| Source tables / dependencies | session.session; access.actor_persona; access.persona_permission; access.permission; registry.menu_item; registry.service_registry; health_url monitoring |
| Backend flow | Resolve active session and active actor_persona assignment. Find VIEW permissions linked to menu_item, then service_registry. Include only customer-facing menu entries permitted by the persona. Check health_url or use fresh monitor result to set availability. |
| Frontend response | Home shortcuts and All Services cards use service_name/description and menu_item.label/page_path. Unavailable services remain visible but disabled. Empty result returns data: []. |
| Safe to repeat? | GET is safe to repeat; short cache may be used if approved. |
| Open points / owner decisions | Wireframe v2 refers to user_service_access, which is absent from . Confirm the final visibility rule is persona_permission → permission → menu_item, and confirm health-check freshness/cache. v5.3 |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/registry/v1/me/services",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


<!-- PDF page 18 -->

```json
{
  "data": [
    {
      "service_id": "d1000000-0000-4000-8000-000000000001",
      "service_name": "Fund Transfer",
      "description": "Send money by UPI, IMPS, NEFT or RTGS",
      "menu_items": [
        {
          "menu_item_id": "e1000000-0000-4000-8000-000000000010",
          "menu_key": "FUND_TRANSFER",
          "label": "Fund Transfer",
          "page_path": "/transfer",
          "sort_order": 1
        }
      ],
      "available": true,
      "health": "UP"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 20
  },
  "meta": {
    "request_id": "req_G8_005",
    "trace_id": "trace_G8_005",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "registry-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_05",
    "trace_id": "trace_ERR_05",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


<!-- PDF page 19 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 19](G8_Platform_API_Schema_assets/workflow_page_19.png)


<!-- PDF page 20 -->

## STEP 1 | API CARD 06 — Read service status


**Data Model table(s): service_registry + health_url monitoring**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Staff/reviewer opens the Status page. |
| Method + endpoint | GET /registry/v1/status |
| Owner microservice | registry-service |
| Who can call | STAFF or REVIEWER with status-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Optional owner_group/status filters; cursor/limit. Health result is UP or DOWN from monitoring, not a stored service_registry column. |
| Source tables / dependencies | registry.service_registry; registry.domain_group; external health_url check / 8C monitoring |
| Backend flow | Read service_registry; join domain_group; obtain latest health probe from monitoring. Keep service status (PLANNED/IN_BUILD/LIVE/DISABLED/RETIRED) distinct from health (UP/DOWN). |
| Frontend response | Show service name, owner group, contract version, registry status and health. Down services show “Temporarily unavailable”; do not hide them. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm monitoring ownership, health freshness threshold and exact response fields because health is not stored in service_registry. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/registry/v1/status",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


```json
{
  "data": [
    {
      "service_id": "d1000000-0000-4000-8000-000000000003",
      "owner_group": "G8",
      "service_name": "Session Service",
      "contract_version": "1.0.0",
      "status": "LIVE",
      "health": "UP",
      "last_checked_at": "2026-11-10T11:59:50Z"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 20
  },
  "meta": {
    "request_id": "req_G8_006",
    "trace_id": "trace_G8_006",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "registry-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 21 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_06",
    "trace_id": "trace_ERR_06",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 21](G8_Platform_API_Schema_assets/workflow_page_21.png)


<!-- PDF page 22 -->

## STEP 1 | API CARD 07 — List menu items for a service


**Data Model table(s): menu_item**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Staff opens Services & Menus administration and selects a service. |
| Method + endpoint | GET /registry/v1/menu-items?service_id={service_id} |
| Owner microservice | registry-service |
| Who can call | STAFF with menu-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Required service_id UUID; optional parent_item_id, status, cursor and limit. status ACTIVE, HIDDEN. ∈ |
| Source tables / dependencies | registry.menu_item; registry.service_registry |
| Backend flow | Validate service_id; read menu_item rows for that service ordered by sort_order. Parent-child references must stay within the same service. |
| Frontend response | Render menu tree with menu_key, label, page_path, sort_order and status. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether hidden menu items are included by default for staff. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/registry/v1/menu-items?service_id={service_id}",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "service_id": "d1000000-0000-4000-8000-000000000001",
      "limit": 50
    }
  }
}
```


### Success response example (JSON)


```json
{
  "data": [
    {
      "menu_item_id": "e1000000-0000-4000-8000-000000000010",
      "service_id": "d1000000-0000-4000-8000-000000000001",
      "parent_item_id": null,
      "menu_key": "FUND_TRANSFER",
      "label": "Fund Transfer",
      "page_path": "/transfer",
      "sort_order": 1,
      "status": "ACTIVE"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 50
  },
  "meta": {
    "request_id": "req_G8_007",
    "trace_id": "trace_G8_007",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "registry-service"
  }
}
```


<!-- PDF page 23 -->

### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_07",
    "trace_id": "trace_ERR_07",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 23](G8_Platform_API_Schema_assets/workflow_page_23.png)


<!-- PDF page 24 -->

## STEP 1 | API CARD 08 — Create a menu item


**Data Model table(s): menu_item**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | Authorised staff adds a feature or submenu to a registered service. |  |
| Method + endpoint | POST /registry/v1/menu-items |  |
| Owner microservice | registry-service |  |
| Who can call | STAFF with menu-create permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Required service_id, menu_key, label, sort_order; optional parent_item_id and page_path. service_id must exist. If parent_item_id is supplied, it must belong to the same service. menu_key is unique. |  |
| Source tables / dependencies | registry.menu_item; registry.service_registry; access.actor; gateway.audit_event |  |
| Backend flow | V | Validate service and parent; enforce unique menu_key and same-service parent; generate menu_item_id; set status default ACTIVE and audit fields. |
| Frontend response | Add the item to the menu tree at sort_order; refresh navigation cache if configured. |  |
| Safe to repeat? | Idempotent with same key and body. |  |
| Open points / owner decisions | Confirm whether menu_key uniqueness should be global or per service; workbook currently declares it globally unique. |  |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/registry/v1/menu-items",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "service_id": "d1000000-0000-4000-8000-000000000001",
    "parent_item_id": "e1000000-0000-4000-8000-000000000010",
    "menu_key": "FUND_TRANSFER_IMPS",
    "label": "IMPS Transfer",
    "page_path": "/transfer/imps",
    "sort_order": 2
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "menu_item_id": "e1000000-0000-4000-8000-000000000011",
    "service_id": "d1000000-0000-4000-8000-000000000001",
    "parent_item_id": "e1000000-0000-4000-8000-000000000010",
    "menu_key": "FUND_TRANSFER_IMPS",
    "label": "IMPS Transfer",
    "page_path": "/transfer/imps",
    "sort_order": 2,
    "status": "ACTIVE",
    "created_at": "2026-11-10T12:00:00Z"
  }
}
```


### Error response example (HTTP 409 Conflict)


<!-- PDF page 25 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "The menu key is already in use.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_08",
    "trace_id": "trace_ERR_08",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 25](G8_Platform_API_Schema_assets/workflow_page_25.png)


<!-- PDF page 26 -->

## STEP 1 | API CARD 09 — Update a menu item


**Data Model table(s): menu_item**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | Staff edits a menu label, page path, parent or display order. |  |
| Method + endpoint | PATCH /registry/v1/menu-items/{menu_item_id} |  |
| Owner microservice | registry-service |  |
| Who can call | STAFF with menu-update permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Path menu_item_id UUID. Body may contain label, page_path, parent_item_id, sort_order or status. Validate parent belongs to same service and no cycles are introduced. |  |
| Source tables / dependencies | registry.menu_item; registry.service_registry; access.actor; gateway.audit_event |  |
| Backend flow |  | Load menu item; validate only allowed fields; enforce same-service parent and no circular hierarchy; update updated_at/updated_by. |
| Frontend response | Refresh affected menu branch. If validation fails, preserve previous menu configuration. |  |
| Safe to repeat? | Replay only with same Idempotency-Key and identical body. |  |
| Open points / owner decisions | Confirm whether status changes or path changes require maker-checker change_request approval. |  |


### Request example


```json
{
  "method": "PATCH",
  "endpoint": "/registry/v1/menu-items/{menu_item_id}",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "menu_item_id": "e1000000-0000-4000-8000-000000000011",
    "label": "IMPS Money Transfer",
    "sort_order": 3
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "menu_item_id": "e1000000-0000-4000-8000-000000000011",
    "service_id": "d1000000-0000-4000-8000-000000000001",
    "parent_item_id": "e1000000-0000-4000-8000-000000000010",
    "menu_key": "FUND_TRANSFER_IMPS",
    "label": "IMPS Money Transfer",
    "page_path": "/transfer/imps",
    "sort_order": 3,
    "status": "ACTIVE",
    "updated_at": "2026-11-10T12:05:00Z"
  }
}
```


### Error response example (HTTP 404 Not Found)


<!-- PDF page 27 -->

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "The menu item was not found.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_09",
    "trace_id": "trace_ERR_09",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 27](G8_Platform_API_Schema_assets/workflow_page_27.png)


<!-- PDF page 28 -->

## STEP 2 | API CARD 10 — List personas


**Data Model table(s): persona**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Staff opens People & Roles to review available personas. |
| Method + endpoint | GET /access/v1/personas |
| Owner microservice | access-control-service |
| Who can call | STAFF with persona-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Optional persona_type, owner_group and status filters; cursor/limit. persona_type EMPLOYEE, CUSTOMER, ADMIN; status ACTIVE, ∈ ∈ INACTIVE. |
| Source tables / dependencies | access.persona; registry.domain_group |
| Backend flow | Read persona rows and join owner_group to domain_group. Do not include actor-specific data in this endpoint. |
| Frontend response | Populate persona catalogue with persona_type, persona_name, owner_group, description and status. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether customer personas are visible to staff from every owner group. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/access/v1/personas",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


```json
{
  "data": [
    {
      "persona_id": "b1000000-0000-4000-8000-000000000003",
      "persona_type": "EMPLOYEE",
      "persona_name": "Advisor",
      "owner_group": "G7",
      "description": "Insurance advisor who services customer policies",
      "status": "ACTIVE"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 20
  },
  "meta": {
    "request_id": "req_G8_010",
    "trace_id": "trace_G8_010",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "access-control-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 29 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_10",
    "trace_id": "trace_ERR_10",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 29](G8_Platform_API_Schema_assets/workflow_page_29.png)


<!-- PDF page 30 -->

## STEP 2 | API CARD 11 — Create a persona


**Data Model table(s): persona**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | Authorised admin creates a new reusable persona definition. |  |
| Method + endpoint | POST /access/v1/personas |  |
| Owner microservice | access-control-service |  |
| Who can call | STAFF with persona-create permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Required persona_type, persona_name, description; optional owner_group. status defaults ACTIVE. owner_group must reference domain_group.group_id. |  |
| Source tables / dependencies | access.persona; registry.domain_group; access.actor |  |
| Backend flow | V | Validate enum, unique persona_name and owner_group FK; generate persona_id; populate created_at/created_by. |
| Frontend response | Show the new persona in role assignment and permission configuration screens. |  |
| Safe to repeat? | Idempotent with same key/body. |  |
| Open points / owner decisions | Confirm persona_name uniqueness case sensitivity and whether inactive personas can be created directly. |  |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/access/v1/personas",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "persona_type": "EMPLOYEE",
    "persona_name": "Branch Operations Officer",
    "owner_group": "G2",
    "description": "Staff persona for branch operations",
    "status": "ACTIVE"
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "persona_id": "b1000000-0000-4000-8000-000000000005",
    "persona_type": "EMPLOYEE",
    "persona_name": "Branch Operations Officer",
    "owner_group": "G2",
    "description": "Staff persona for branch operations",
    "status": "ACTIVE",
    "created_at": "2026-11-10T12:00:00Z"
  }
}
```


### Error response example (HTTP 409 Conflict)


<!-- PDF page 31 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "A persona with this unique name already exists.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_11",
    "trace_id": "trace_ERR_11",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 31](G8_Platform_API_Schema_assets/workflow_page_31.png)


<!-- PDF page 32 -->

## STEP 2 | API CARD 12 — List permissions


**Data Model table(s): permission**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Staff opens the permission catalogue for a menu item or service. |
| Method + endpoint | GET /access/v1/permissions |
| Owner microservice | access-control-service |
| Who can call | STAFF with permission-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Optional menu_item_id, action and status filters; cursor/limit. action ∈ VIEW, CREATE, EDIT, DELETE, APPROVE. |
| Source tables / dependencies | access.permission; registry.menu_item |
| Backend flow | Read permission rows; join menu_item to show menu_key/label. Use exact permission_code and action values. |
| Frontend response | Populate permission catalogue; show permission_code, menu item, action, description and status. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether permission_code naming convention is enforced beyond global uniqueness. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/access/v1/permissions",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "menu_item_id": "e1000000-0000-4000-8000-000000000010",
      "limit": 20
    }
  }
}
```


### Success response example (JSON)


```json
{
  "data": [
    {
      "permission_id": "c1000000-0000-4000-8000-000000000001",
      "permission_code": "FUND_TRANSFER.VIEW",
      "menu_item_id": "e1000000-0000-4000-8000-000000000010",
      "action": "VIEW",
      "description": "See the Fund Transfer page",
      "status": "ACTIVE"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 20
  },
  "meta": {
    "request_id": "req_G8_012",
    "trace_id": "trace_G8_012",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "access-control-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 33 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_12",
    "trace_id": "trace_ERR_12",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 33](G8_Platform_API_Schema_assets/workflow_page_33.png)


<!-- PDF page 34 -->

## STEP 2 | API CARD 13 — Create a permission


**Data Model table(s): permission**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | Authorised administrator defines an action permission for a menu item. |  |
| Method + endpoint | POST /access/v1/permissions |  |
| Owner microservice | access-control-service |  |
| Who can call | STAFF with permission-create permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Required permission_code, menu_item_id, action, description. action ∈ VIEW, CREATE, EDIT, DELETE, APPROVE; status defaults ACTIVE. |  |
| Source tables / dependencies | access.permission; registry.menu_item; access.actor |  |
| Backend flow | V | Validate menu_item FK, action enum and unique permission_code; generate permission_id; populate created_at/created_by. |
| Frontend response | Permission becomes available for persona_permission mapping after creation. |  |
| Safe to repeat? | Idempotent with same key/body. |  |
| Open points / owner decisions | Confirm if permission deactivation should use status INACTIVE or require a change_request. |  |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/access/v1/permissions",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "permission_code": "FUND_TRANSFER.CREATE",
    "menu_item_id": "e1000000-0000-4000-8000-000000000010",
    "action": "CREATE",
    "description": "Send money",
    "status": "ACTIVE"
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "permission_id": "c1000000-0000-4000-8000-000000000006",
    "permission_code": "FUND_TRANSFER.CREATE",
    "menu_item_id": "e1000000-0000-4000-8000-000000000010",
    "action": "CREATE",
    "description": "Send money",
    "status": "ACTIVE",
    "created_at": "2026-11-10T12:00:00Z"
  }
}
```


### Error response example (HTTP 409 Conflict)


<!-- PDF page 35 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "A permission with this unique code already exists.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_13",
    "trace_id": "trace_ERR_13",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 35](G8_Platform_API_Schema_assets/workflow_page_35.png)


<!-- PDF page 36 -->

## STEP 2 | API CARD 14 — Read permissions assigned to a persona


**Data Model table(s): persona_permission**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Staff opens a persona and reviews its allowed actions. |
| Method + endpoint | GET /access/v1/personas/{persona_id}/permissions |
| Owner microservice | access-control-service |
| Who can call | STAFF with persona-permission read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Path persona_id UUID; optional status filter. Return only active persona_permission links and active permission rows unless staff asks for inactive mappings. |
| Source tables / dependencies | access.persona; access.persona_permission; access.permission; registry.menu_item |
| Backend flow | Read persona_permission by persona_id; join permission and menu_item. Enforce active persona and permission statuses. |
| Frontend response | Show each permission code, menu item, action and link creation metadata. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether the response should include inactive permission links for audit. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/access/v1/personas/{persona_id}/permissions",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "path": {
      "persona_id": "b1000000-0000-4000-8000-000000000001"
    }
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "persona_id": "b1000000-0000-4000-8000-000000000001",
    "persona_name": "Individual Customer",
    "permissions": [
      {
        "permission_id": "c1000000-0000-4000-8000-000000000001",
        "permission_code": "FUND_TRANSFER.VIEW",
        "menu_item_id": "e1000000-0000-4000-8000-000000000010",
        "menu_key": "FUND_TRANSFER",
        "label": "Fund Transfer",
        "action": "VIEW"
      }
    ]
  },
  "meta": {
    "request_id": "req_G8_014",
    "trace_id": "trace_G8_014",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "access-control-service"
  }
}
```


<!-- PDF page 37 -->

### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_14",
    "trace_id": "trace_ERR_14",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 37](G8_Platform_API_Schema_assets/workflow_page_37.png)


<!-- PDF page 38 -->

## STEP 2 | API CARD 15 — Assign a permission to a persona


**Data Model table(s): persona_permission**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | Admin grants an existing permission to an existing persona. |  |
| Method + endpoint | PUT /access/v1/personas/{persona_id}/permissions/{permission_id} |  |
| Owner microservice | access-control-service |  |
| Who can call | STAFF with persona-permission manage permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Path persona_id and permission_id UUIDs. No body required. Both referenced rows must exist and be ACTIVE. |  |
| Source tables / dependencies | access.persona_permission; access.persona; access.permission; access.actor |  |
| Backend flow |  | Insert the composite-key link (persona_id, permission_id); reject duplicate mapping; populate created_at/created_by. |
| Frontend response | Refresh effective permissions for future requests; active sessions may need permission re-evaluation at each request. |  |
| Safe to repeat? | Idempotent; repeating the same assignment returns the existing mapping. |  |
| Open points / owner decisions | Confirm whether assignment/removal requires a Step 7 change_request and how permission cache is invalidated. |  |


### Request example


```json
{
  "method": "PUT",
  "endpoint": "/access/v1/personas/{persona_id}/permissions/{permission_id}",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "path": {
      "persona_id": "b1000000-0000-4000-8000-000000000001",
      "permission_id": "c1000000-0000-4000-8000-000000000001"
    }
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "persona_id": "b1000000-0000-4000-8000-000000000001",
    "permission_id": "c1000000-0000-4000-8000-000000000001",
    "created_at": "2026-11-10T12:00:00Z",
    "created_by": "a1000000-0000-4000-8000-000000000901"
  }
}
```


### Error response example (HTTP 409 Conflict)


<!-- PDF page 39 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "This persona already has the selected permission.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_15",
    "trace_id": "trace_ERR_15",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 39](G8_Platform_API_Schema_assets/workflow_page_39.png)


<!-- PDF page 40 -->

## STEP 3 | API CARD 16 — List configured API routes


**Data Model table(s): route**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Platform Admin opens the API route catalogue. |
| Method + endpoint | GET /gateway/v1/routes |
| Owner microservice | gateway-routing-service |
| Who can call | STAFF with route-read permission; SERVICE for approved discovery only |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt; or trusted SERVICE identity; Accept: application/json; X-Request-ID. |
| Inputs and validation | Optional service_id, method, auth_required and token_realm filters; cursor/limit. method GET, POST, PATCH, DELETE; token_realm ∈ ∈ CUSTOMER, STAFF, ANY. |
| Source tables / dependencies | gateway.route; registry.service_registry; access.permission |
| Backend flow | Read route rows and join service_registry and permission. Return route_id, service_id, permission_id, method, path, auth_required, safe_to_repeat, timeout_ms, token_realm, counts_as_activity. |
| Frontend response | Populate API catalogue and show route ownership and gateway policy metadata. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether route paths are unique by (method,path), and whether a route can have auth_required=true with permission_id null. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/gateway/v1/routes",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "service_id": "d1000000-0000-4000-8000-000000000003",
      "limit": 50
    }
  }
}
```


### Success response example (JSON)


<!-- PDF page 41 -->

```json
{
  "data": [
    {
      "route_id": "f1000000-0000-4000-8000-000000000001",
      "service_id": "d1000000-0000-4000-8000-000000000003",
      "permission_id": null,
      "method": "POST",
      "path": "/sessions",
      "auth_required": false,
      "safe_to_repeat": false,
      "timeout_ms": 3000,
      "token_realm": "ANY",
      "counts_as_activity": false
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 50
  },
  "meta": {
    "request_id": "req_G8_016",
    "trace_id": "trace_G8_016",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "gateway-routing-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_16",
    "trace_id": "trace_ERR_16",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


<!-- PDF page 42 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 42](G8_Platform_API_Schema_assets/workflow_page_42.png)


<!-- PDF page 43 -->

## STEP 3 | API CARD 17 — Read a configuration setting


**Data Model table(s): config_setting**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | A service or authorised administrator requests a named runtime setting. |
| Method + endpoint | GET /internal/v1/settings/{config_key} |
| Owner microservice | configuration-service (Group 8; proposed microservice name) |
| Who can call | SERVICE or STAFF with setting-read permission |
| Required headers | Trusted SERVICE identity or Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Path config_key required. Optional service_id, condition_field and condition_value resolve scoped/conditional values. Return only ACTIVE and effective versions. |
| Source tables / dependencies | config.config_setting; registry.service_registry |
| Backend flow | Resolve exact key and applicable service/condition scope; select one ACTIVE row effective at the current bank time. Never return secrets or unrelated settings. |
| Frontend response | Calling service receives value, value_type, unit, version, source and effective period. |
| Safe to repeat? | GET is safe to repeat; short cache only if configured. |
| Open points / owner decisions | Confirm deterministic precedence when global and service-specific conditional rows both match; confirm secret-setting policy. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/internal/v1/settings/{config_key}",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "path": {
      "config_key": "SESSION_IDLE_TIMEOUT"
    },
    "query": {
      "condition_field": "g1_customer_type",
      "condition_value": "INDIVIDUAL"
    }
  }
}
```


### Success response example (JSON)


<!-- PDF page 44 -->

```json
{
  "data": {
    "config_id": "1c000000-0000-4000-8000-000000000001",
    "config_key": "SESSION_IDLE_TIMEOUT",
    "category": "SESSION",
    "service_id": null,
    "condition_field": null,
    "condition_value": null,
    "value": "15",
    "value_type": "DURATION",
    "unit": "minutes",
    "source": "PLATFORM",
    "source_ref": null,
    "version": 1,
    "status": "ACTIVE",
    "effective_from": "2026-11-01T00:00:00Z",
    "effective_to": null
  },
  "meta": {
    "request_id": "req_G8_017",
    "trace_id": "trace_G8_017",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "configuration-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_17",
    "trace_id": "trace_ERR_17",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


<!-- PDF page 45 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 45](G8_Platform_API_Schema_assets/workflow_page_45.png)


<!-- PDF page 46 -->

## STEP 3 | API CARD 18 · Read simulated bank


date and time


**Data Model table(s): bank_clock**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | A service or staff page needs the shared bank date/time. |
| Method + endpoint | GET /clock/v1/today |
| Owner microservice | bank-clock-service |
| Who can call | SERVICE or STAFF |
| Required headers | Trusted SERVICE identity or Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | No body. The model describes one row with clock_id MAIN. |
| Source tables / dependencies | clock.bank_clock |
| Backend flow | Read the singleton MAIN row; return bank_date, bank_time, mode and updated_at. Use bank clock, not server local time, for simulated business decisions. |
| Frontend response | All services display/compute against the same simulated bank time. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether API returns bank timezone explicitly; the model stores date and time separately and does not specify timezone. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/clock/v1/today",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


```json
{
  "data": {
    "clock_id": "MAIN",
    "bank_date": "2026-11-10",
    "bank_time": "12:00:00",
    "mode": "DEMO",
    "updated_at": "2026-11-10T06:30:00Z"
  },
  "meta": {
    "request_id": "req_G8_019",
    "trace_id": "trace_G8_019",
    "timestamp": "2026-11-10T06:30:00Z",
    "simulated": true,
    "source": "bank-clock-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 47 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_19",
    "trace_id": "trace_ERR_19",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 47](G8_Platform_API_Schema_assets/workflow_page_47.png)


<!-- PDF page 48 -->

## STEP 3 | API CARD 19 · Read business calendar


**Data Model table(s): business_calendar**


| Card field |  | Detailed contract |  |
| --- | --- | --- | --- |
| Card field v5 3 correction: this API returns working day and holiday data only Pa | ay | Detailed contract yment rail cut offs are Group 4 owned and are not columns in |  |
| Customer action / screen v5.3 correction: this API returns working-day and holiday data only. Pa business_calendar. Customer action / screen | ay | ym A | A service requests working days, holidays and for a date range. ment rail cut-offs are Group 4-owned and are not columns in A service requests working days and holidays for a date range. A service requests working days and holidays for a date range. |
| Method + endpoint |  | GET /clock/v1/calendar |  |
| p |  |  |  |
| Owner microservice |  | bank-clock-service |  |
| Who can call |  | SERVICE or STAFF |  |
| Required headers |  | Trusted SERVICE identity or Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |  |
| Inputs and validation |  | Required from and to ISO dates; from must be ≤ to. Limit date range to configured horizon; cursor optional for long ranges. |  |
| Source tables / dependencies |  | clock.business_calendar |  |
| Backend flow |  |  | Read one row per calendar_date; return is_working_day, holiday_name, and . Null cut-off is allowed on non-working days. Read one row per calendar_date; return calendar_date, is_working_day and holiday_name. No payment rail cut-off fields are stored in this table. |
| Frontend response |  | Services use a shared calendar instead of maintaining separate holiday lists. |  |
| Safe to repeat? |  | GET is safe to repeat. |  |
| Open points / owner decisions |  | Confirm supported calendar horizon, timezone convention and whether missing dates are an error or treated as unknown. |  |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/clock/v1/calendar",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "from": "2026-11-09",
      "to": "2026-11-16"
    }
  }
}
```


### Success response example (JSON)


<!-- PDF page 49 -->

<!-- PDF page 49 contains an overlapping, malformed draft of the calendar JSON example; the complete valid version follows. -->


```json
{
  "data": [
    {
      "calendar_date": "2026-11-09",
      "is_working_day": true,
      "holiday_name": null
    },
    {
      "calendar_date": "2026-11-14",
      "is_working_day": false,
      "holiday_name": "Second Saturday (sample)"
    }
  ],
  "page": {"next_cursor": null, "limit": 20},
  "meta": {
    "request_id": "req_G8_020",
    "trace_id": "trace_G8_020",
    "timestamp": "2026-11-10T06:30:00Z",
    "simulated": true,
    "source": "bank-clock-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_20",
    "trace_id": "trace_ERR_20",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


Correct v5.3 response fields: calendar_date, is_working_day, holiday_name. updated_at and updated_by are database metadata and
may be included only for authorised administrative views.


<!-- PDF page 50 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 50](G8_Platform_API_Schema_assets/workflow_page_50.png)


<!-- PDF page 51 -->

## STEP 3 | API CARD 20 · Advance simulated bank


date/time


**Data Model table(s): bank_clock + business_calendar**


| Card field | Detailed contract |  |
| --- | --- | --- |
| Customer action / screen | An authorised staff controller advances the bank clock in DEMO mode. |  |
| Method + endpoint | POST /clock/v1/advance |  |
| Owner microservice | bank-clock-service |  |
| Who can call | STAFF with clock-control permission |  |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |  |
| Inputs and validation | Body bank_date and bank_time required. Only mode DEMO; requested combined timestamp must be later than current bank clock. |  |
| Source tables / dependencies | clock.bank_clock; clock.business_calendar; access.actor; gateway.idempotency_record; gateway.audit_event |  |
| Backend flow | Lock MAIN row; validate DEMO mode and forward-only movement; update bank_date, bank_time, updated_at, updated_by. Record idempotency and audit. |  |
|  |  | Lock MAIN row; validate DEMO mode and forward-only movement; update bank_date, bank_time, updated_at, updated_by. Record idempotency and audit. |
| Frontend response | Return new shared clock value. All services read it through GET /clock/v1/today. |  |
| Safe to repeat? | Use Idempotency-Key; same key/body returns original result. |  |
| Open points / owner decisions | Confirm who may advance clock, whether calendar date must exist and whether time may be advanced within the same date. |  |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/clock/v1/advance",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "bank_date": "2026-11-11",
    "bank_time": "00:00:00"
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "clock_id": "MAIN",
    "bank_date": "2026-11-11",
    "bank_time": "00:00:00",
    "mode": "DEMO",
    "updated_at": "2026-11-10T06:35:00Z",
    "updated_by": "a1000000-0000-4000-8000-000000000901"
  },
  "meta": {
    "request_id": "req_G8_021",
    "trace_id": "trace_G8_021",
    "timestamp": "2026-11-10T06:35:00Z",
    "simulated": true,
    "source": "bank-clock-service"
  }
}
```


### Error response example (HTTP 409 Conflict)


v5.3 alignment: bank_clock is master data. This command must be a Step 7 maker-checker proposal unless the Data Model owner formally
approves a DEMO-mode operational exception.


<!-- PDF page 52 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "The bank clock is LIVE or the requested time is not later than the current value.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_21",
    "trace_id": "trace_ERR_21",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 52](G8_Platform_API_Schema_assets/workflow_page_52.png)


<!-- PDF page 53 -->

## STEP 3 | API CARD 21 · Check service health


**Data Model table(s): service_registry.health_url + route**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Gateway or monitoring service probes a registered health_url. |
| Method + endpoint | GET /health |
| Owner microservice | Health endpoint of each registered service |
| Who can call | SERVICE (trusted monitoring/Gateway identity) |
| Required headers | Accept: application/json; X-Request-ID. No customer session token. |
| Inputs and validation | No body. Health response status is UP or DOWN; health is not a persisted service_registry column. |
| Source tables / dependencies | registry.service_registry.health_url; gateway.audit_event (only if audit policy requires) |
| Backend flow | Probe the service's configured health_url; enforce timeout; return minimal health status. Do not use this endpoint as a substitute for business API readiness. |
| Frontend response | Status page shows health separately from service_registry.status. |
| Safe to repeat? | Read-only; safe to repeat. |
| Open points / owner decisions | Confirm common health response schema and monitoring cadence with the platform operations owner. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/health",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


```json
{
  "data": {
    "status": "UP",
    "service_id": "d1000000-0000-4000-8000-000000000003",
    "checked_at": "2026-11-10T06:40:00Z"
  },
  "meta": {
    "request_id": "req_G8_022",
    "trace_id": "trace_G8_022",
    "timestamp": "2026-11-10T06:40:00Z",
    "simulated": true,
    "source": "health-monitor"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_22",
    "trace_id": "trace_ERR_22",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


<!-- PDF page 54 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 54](G8_Platform_API_Schema_assets/workflow_page_54.png)


<!-- PDF page 55 -->

## STEP 4 | API CARD 22 · List Platform actors


**Data Model table(s): actor**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Admin opens the people directory to review login identities already known to the Platform. |
| Method + endpoint | GET /access/v1/actors |
| Owner microservice | access-control-service |
| Who can call | STAFF with actor-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Optional status, persona_id and identity-type filters; cursor/limit. Do not accept user_id/party_id from an untrusted customer caller. |
| Source tables / dependencies | access.actor; access.actor_persona; access.persona |
| Backend flow | Read actor rows and optionally join active persona assignments. Mask or omit external identifiers unless required by the admin role. |
| Frontend response | Populate people directory with actor_id, user_id, party_id/employee_id as permitted, status and active personas. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm which staff roles may see Group 1 user_id, party_id and employee_id; these are plain values, not Platform-owned identities. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/access/v1/actors",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "status": "ACTIVE",
      "limit": 20
    }
  }
}
```


### Success response example (JSON)


```json
{
  "data": [
    {
      "actor_id": "a1000000-0000-4000-8000-000000000012",
      "user_id": "U-30012",
      "party_id": "P-10012",
      "employee_id": null,
      "status": "ACTIVE",
      "created_at": "2026-11-06T03:42:00Z"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 20
  },
  "meta": {
    "request_id": "req_G8_023",
    "trace_id": "trace_G8_023",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "access-control-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 56 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_23",
    "trace_id": "trace_ERR_23",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 56](G8_Platform_API_Schema_assets/workflow_page_56.png)


<!-- PDF page 57 -->

## STEP 4 | API CARD 23 · List personas assigned t


an actor


**Data Model table(s): actor_persona**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Staff opens a person’s role/persona assignment history. |
| Method + endpoint | GET /access/v1/actors/{actor_id}/personas |
| Owner microservice | access-control-service |
| Who can call | STAFF with actor-persona read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Path actor_id UUID; optional status and effective_at filters. status ∈ PENDING_APPROVAL, ACTIVE, REVOKED. |
| Source tables / dependencies | access.actor; access.actor_persona; access.persona |
| Backend flow | Read actor_persona links for actor_id; join persona; return assignment status, valid_from/to, reason and approval/revocation metadata. |
| Frontend response | Show active, pending and revoked persona assignments distinctly. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether requester can view only current assignments or full historical assignments. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/access/v1/actors/{actor_id}/personas",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "path": {
      "actor_id": "a1000000-0000-4000-8000-000000000031"
    }
  }
}
```


### Success response example (JSON)


```json
{
  "data": [
    {
      "actor_persona_id": "a2000000-0000-4000-8000-000000000002",
      "actor_id": "a1000000-0000-4000-8000-000000000031",
      "persona_id": "b1000000-0000-4000-8000-000000000003",
      "persona_type": "EMPLOYEE",
      "persona_name": "Advisor",
      "status": "ACTIVE",
      "valid_from": "2026-11-06T06:30:00Z",
      "valid_to": null,
      "reason": "Joined the G7 advisor team"
    }
  ],
  "meta": {
    "request_id": "req_G8_024",
    "trace_id": "trace_G8_024",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "access-control-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 58 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_24",
    "trace_id": "trace_ERR_24",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 58](G8_Platform_API_Schema_assets/workflow_page_58.png)


<!-- PDF page 59 -->

## STEP 4 | API CARD 24 · Request a persona


assignment


**Data Model table(s): actor_persona**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Authorised staff requests assigning an existing persona to an actor. |
| Method + endpoint | POST /access/v1/actors/{actor_id}/personas |
| Owner microservice | access-control-service |
| Who can call | STAFF with assignment-request permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |
| Inputs and validation | Path actor_id UUID. Body persona_id, valid_from and optional valid_to/reason. requested_by derived from authenticated actor. Initial status PENDING_APPROVAL unless bootstrap exception approved. |
| Source tables / dependencies | access.actor; access.persona; access.actor_persona; access.actor (requested_by); gateway.idempotency_record; gateway.audit_event |
| Backend flow | Validate actor/persona active, date range and no conflicting active assignment; insert actor_persona with PENDING_APPROVAL and requested_by; do not activate permissions before approval. |
| Frontend response | Show request as Pending approval. Actor does not gain the persona until checker approves. |
| Safe to repeat? | Idempotent with same key/body. |
| Open points / owner decisions | Confirm whether every assignment requires maker-checker or if bootstrap/default persona is exempt; define duplicate active assignment constraint. |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/access/v1/actors/{actor_id}/personas",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "persona_id": "b1000000-0000-4000-8000-000000000003",
    "valid_from": "2026-11-11T00:00:00Z",
    "reason": "Temporary advisor coverage"
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "actor_persona_id": "a2000000-0000-4000-8000-000000000005",
    "actor_id": "a1000000-0000-4000-8000-000000000031",
    "persona_id": "b1000000-0000-4000-8000-000000000003",
    "status": "PENDING_APPROVAL",
    "valid_from": "2026-11-11T00:00:00Z",
    "valid_to": null,
    "reason": "Temporary advisor coverage",
    "requested_by": "a1000000-0000-4000-8000-000000000902",
    "created_at": "2026-11-10T12:00:00Z"
  }
}
```


### Error response example (HTTP 409 Conflict)


<!-- PDF page 60 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "An equivalent active or pending assignment already exists.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_25",
    "trace_id": "trace_ERR_25",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 60](G8_Platform_API_Schema_assets/workflow_page_60.png)


<!-- PDF page 61 -->

## STEP 4 | API CARD 25 · Approve or reject a


persona assignment


**Data Model table(s): actor_persona**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | A different authorised checker reviews a pending actor_persona assignment. |
| Method + endpoint | POST /access/v1/actor-persona-assignments/{actor_persona_id}/decision |
| Owner microservice | access-control-service |
| Who can call | STAFF checker with assignment-approve permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |
| Inputs and validation | Path actor_persona_id UUID. Body decision APPROVE, REJECT; ∈ optional decision_note. checker_id derived from caller and must differ from requested_by/maker. |
| Source tables / dependencies | access.actor_persona; access.actor; access.persona; gateway.audit_event; gateway.idempotency_record |
| Backend flow | Load pending row; enforce maker-checker separation and current status; on approve set status ACTIVE and approved_by; on reject set status REVOKED or reject without activation (owner must confirm lifecycle semantics); preserve history. |
| Frontend response | Show Approved or Rejected status. Recompute effective permissions after approval. |
| Safe to repeat? | Use Idempotency-Key; repeated same decision returns current state. |
| Open points / owner decisions | Data Model actor_persona has PENDING_APPROVAL, ACTIVE, REVOKED but no REJECTED. Confirm whether rejection should set REVOKED or add a distinct status. |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/access/v1/actor-persona-assignments/{actor_persona_id}/decision",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "decision": "APPROVE",
    "decision_note": "Approved for temporary coverage."
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "actor_persona_id": "a2000000-0000-4000-8000-000000000005",
    "actor_id": "a1000000-0000-4000-8000-000000000031",
    "persona_id": "b1000000-0000-4000-8000-000000000003",
    "status": "ACTIVE",
    "approved_by": "a1000000-0000-4000-8000-000000000901",
    "valid_from": "2026-11-11T00:00:00Z"
  }
}
```


### Error response example (HTTP 409 Conflict)


<!-- PDF page 62 -->

```json
{
  "error": {
    "code": "CONFLICT",
    "message": "The assignment is no longer pending approval.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_26",
    "trace_id": "trace_ERR_26",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 62](G8_Platform_API_Schema_assets/workflow_page_62.png)


<!-- PDF page 63 -->

## STEP 4 | API CARD 26 · Read the current admin


profile


**Data Model table(s): admin_profile**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Admin opens their own profile in the Platform administration area. |
| Method + endpoint | GET /access/v1/admin-profile/me |
| Owner microservice | access-control-service |
| Who can call | STAFF/ADMIN with admin-profile read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | No actor_id in request; derive actor from authenticated session. Return only admin_profile fields. |
| Source tables / dependencies | access.actor; access.admin_profile |
| Backend flow | Resolve actor_id from session and read the one-to-one admin_profile row. Do not return credentials or Group 1 authentication secrets. |
| Frontend response | Show full_name, email, phone and status to the signed-in admin. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm whether profile update is in scope; no admin_profile update card is defined without approved editable fields. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/access/v1/admin-profile/me",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


```json
{
  "data": {
    "actor_id": "a1000000-0000-4000-8000-000000000901",
    "full_name": "Test Admin One",
    "email": "admin.one@mirai-bank.test",
    "phone": "+91-90000-00001",
    "status": "ACTIVE",
    "created_at": "2026-11-02T03:30:00Z"
  },
  "meta": {
    "request_id": "req_G8_027",
    "trace_id": "trace_G8_027",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true,
    "source": "access-control-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 64 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_27",
    "trace_id": "trace_ERR_27",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 64](G8_Platform_API_Schema_assets/workflow_page_64.png)


<!-- PDF page 65 -->

## STEP 5 | API CARD 27 · Start a Platform session


after Group 1 confirms login


**Data Model table(s): actor + actor_persona + session**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Group 1 authentication service calls Platform after successful login and OTP checks. |
| Method + endpoint | POST /sessions/v1/sessions |
| Owner microservice | session-service |
| Who can call | Trusted SERVICE caller: Group 1 only |
| Required headers | Trusted Group 1 service identity; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. No customer token yet. |
| Inputs and validation | Payload contains Group 1 login confirmation fields: user_id, optional party_id/employee_id, persona_type, persona_name or resolved persona, login_ref/auth_session_ref, login_response (redacted), channel and optional device_type/ip_address. Do not accept a raw token. |
| Source tables / dependencies | access.actor; access.actor_persona; access.persona; config.config_setting; session.session; gateway.audit_event; gateway.idempotency_record |
| Backend flow | Validate trusted Group 1 service and successful login_response. Upsert actor by unique user_id; resolve default persona using active config_setting; create/verify actor_persona; create session with 3-column composite FK, copied Group 1 IDs and login_ref. Never persist raw token. |
| Frontend response | Return session_id and session status; the login pass/token mechanism is owned by Group 1/Gateway and must not be stored in this Platform table. |
| Safe to repeat? | Idempotent with Group 1 event/login_ref and Idempotency-Key. |
| Open points / owner decisions | Confirm exact Group 1 callback schema, token/login-pass handoff, default persona resolution, and whether login_response must be redacted before persistence. |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/sessions/v1/sessions",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "user_id": "U-30012",
    "party_id": "P-10012",
    "employee_id": null,
    "persona_type": "CUSTOMER",
    "persona_name": "Individual Customer",
    "login_ref": "g1-asr-7f3a91",
    "login_response": {
      "auth_result": "SUCCESS",
      "realm": "customer",
      "user_id": "U-30012",
      "party_id": "P-10012",
      "employee_id": null,
      "subject_type": "CUSTOMER",
      "customer_type": "INDIVIDUAL",
      "auth_session_ref": "g1-asr-7f3a91",
      "mfa_used": true,
      "authenticated_at": "2026-11-10T09:30:05+05:30"
    },
    "channel": "MOBILE",
    "device_type": "Android phone",
    "ip_address": "10.0.4.27"
  }
}
```


### Success response example (JSON)


<!-- PDF page 66 -->

```json
{
  "data": {
    "session_id": "5e000000-0000-4000-8000-000000000004",
    "actor_id": "a1000000-0000-4000-8000-000000000012",
    "actor_persona_id": "a2000000-0000-4000-8000-000000000001",
    "user_id": "U-30012",
    "party_id": "P-10012",
    "employee_id": null,
    "persona_id": "b1000000-0000-4000-8000-000000000001",
    "persona_type": "CUSTOMER",
    "persona_name": "Individual Customer",
    "login_ref": "g1-asr-7f3a91",
    "status": "ACTIVE",
    "created_at": "2026-11-10T04:00:00Z",
    "last_seen_at": "2026-11-10T04:00:00Z"
  }
}
```


### Error response example (HTTP 409 Conflict)


```json
{
  "error": {
    "code": "CONFLICT",
    "message": "A Platform session already exists for this Group 1 login reference.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_28",
    "trace_id": "trace_ERR_28",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 66](G8_Platform_API_Schema_assets/workflow_page_66.png)


<!-- PDF page 67 -->

## STEP 5 | API CARD 28 · Check the current


Platform session


**Data Model table(s): session + config_setting**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | A signed-in customer or staff member opens/refreshes a page; the session is checked on every protected request. |
| Method + endpoint | GET /sessions/v1/sessions/current |
| Owner microservice | session-service |
| Who can call | CUSTOMER or STAFF with valid login pass |
| Required headers | Authorization: Bearer &lt;login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | No body. No party_id or actor_id supplied by browser. Validate status ACTIVE, idle timeout and maximum duration using active config_setting values. |
| Source tables / dependencies | session.session; config.config_setting; access.actor; access.actor_persona; session.session_sync_event |
| Backend flow | Resolve session from login pass; check actor and persona assignment remains valid; compare current time with last_seen_at and created_at against SESSION_IDLE_TIMEOUT and SESSION_MAX_DURATION. Update last_seen_at only when counts_as_activity policy allows. |
| Frontend response | If active, continue page. If expired/timed out/revoked, clear local login state and show “You have been logged out for your safety. Please log in again.” |
| Safe to repeat? | GET is safe to repeat; activity timestamp update is controlled by route.counts_as_activity. |
| Open points / owner decisions | The wireframe proposes 15-minute idle and 60-minute maximum values; workbook samples show these values but approval is still required. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/sessions/v1/sessions/current",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


```json
{
  "data": {
    "session_id": "5e000000-0000-4000-8000-000000000004",
    "actor_id": "a1000000-0000-4000-8000-000000000012",
    "actor_persona_id": "a2000000-0000-4000-8000-000000000001",
    "party_id": "P-10012",
    "persona_id": "b1000000-0000-4000-8000-000000000001",
    "persona_type": "CUSTOMER",
    "persona_name": "Individual Customer",
    "status": "ACTIVE",
    "last_seen_at": "2026-11-10T04:05:00Z",
    "expires_at": "2026-11-10T05:00:00Z"
  },
  "meta": {
    "request_id": "req_G8_029",
    "trace_id": "trace_G8_029",
    "timestamp": "2026-11-10T04:05:00Z",
    "simulated": true,
    "source": "session-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 68 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_29",
    "trace_id": "trace_ERR_29",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 68](G8_Platform_API_Schema_assets/workflow_page_68.png)


<!-- PDF page 69 -->

## STEP 5 | API CARD 29 · End the current Platform


session (logout)


**Data Model table(s): session + session_sync_event**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Customer selects Log out in the shared header. |
| Method + endpoint | DELETE /sessions/v1/sessions/current |
| Owner microservice | session-service |
| Who can call | CUSTOMER or STAFF with current session |
| Required headers | Authorization: Bearer &lt;login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | No body. Resolve session from login pass; do not accept session_id or party_id from browser. |
| Source tables / dependencies | session.session; session.session_sync_event; gateway.audit_event |
| Backend flow | Set session.status=LOGGED_OUT, ended_at and end_reason=USER_LOGOUT. Create outbound session_sync_event to Group 1 with direction TO_GROUP1 and event_type PLATFORM_SESSION_REVOKED; retry delivery if needed. |
| Frontend response | Clear session and cached state across Platform pages; return user to pre-login Home. Group 1 is notified asynchronously. |
| Safe to repeat? | Idempotent; repeated logout returns already-ended state without creating duplicate side effects. |
| Open points / owner decisions | Confirm Group 1 revocation callback contract and whether logout should be synchronous or event-driven. |


### Request example


```json
{
  "method": "DELETE",
  "endpoint": "/sessions/v1/sessions/current",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {}
}
```


### Success response example (JSON)


```json
{
  "data": {
    "session_id": "5e000000-0000-4000-8000-000000000004",
    "status": "LOGGED_OUT",
    "ended_at": "2026-11-10T04:10:00Z",
    "end_reason": "USER_LOGOUT",
    "group1_sync_status": "PENDING"
  },
  "meta": {
    "request_id": "req_G8_030",
    "trace_id": "trace_G8_030",
    "timestamp": "2026-11-10T04:10:00Z",
    "simulated": true,
    "source": "session-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 70 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_30",
    "trace_id": "trace_ERR_30",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 70](G8_Platform_API_Schema_assets/workflow_page_70.png)


<!-- PDF page 71 -->

## STEP 5 | API CARD 30 · Receive or retry a Group


1 session-sync event


**Data Model table(s): session_sync_event**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Group 1 notifies Platform of logout/revocation, or Platform retries a timeout/revocation notice. |
| Method + endpoint | POST /internal/v1/session-sync-events |
| Owner microservice | session-service |
| Who can call | Trusted SERVICE caller: Group 1 for inbound; Platform worker for outbound retry |
| Required headers | Trusted service identity; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |
| Inputs and validation | Required event_id, direction, event_type, auth_session_ref, reason, correlation_id, occurred_at. direction TO_GROUP1, FROM_GROUP1; ∈ event_type from model enum. |
| Source tables / dependencies | session.session_sync_event; session.session; gateway.audit_event |
| Backend flow | Enforce composite key (event_id,direction); resolve session by auth_session_ref/login_ref; process G1_USER_LOGOUT/G1_SESSION_REVOKED or outbound timeout/revoke event. Track status, group1_result, attempt_count, next_retry_at, last_error, completed_at. |
| Frontend response | No direct customer screen. A completed inbound revocation ends the matching Platform session; failed delivery remains retryable and visible to operations. |
| Safe to repeat? | Idempotent by event_id+direction; duplicate delivery returns recorded result. |
| Open points / owner decisions | Clarify if event_id uniqueness is global or scoped by direction; confirm retry schedule and exact Group 1 response vocabulary. |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/internal/v1/session-sync-events",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "event_id": "g1-evt-55822",
    "direction": "FROM_GROUP1",
    "event_type": "G1_USER_LOGOUT",
    "auth_session_ref": "g1-asr-7f3a91",
    "reason": "USER_LOGOUT",
    "correlation_id": "trc-91c0ff",
    "occurred_at": "2026-11-10T04:10:00Z"
  }
}
```


### Success response example (JSON)


<!-- PDF page 72 -->

```json
{
  "data": {
    "event_id": "g1-evt-55822",
    "direction": "FROM_GROUP1",
    "event_type": "G1_USER_LOGOUT",
    "session_id": "5e000000-0000-4000-8000-000000000004",
    "auth_session_ref": "g1-asr-7f3a91",
    "status": "COMPLETED",
    "group1_result": "ALREADY_TERMINATED",
    "attempt_count": 1,
    "correlation_id": "trc-91c0ff",
    "occurred_at": "2026-11-10T04:10:00Z",
    "completed_at": "2026-11-10T04:10:01Z"
  }
}
```


### Error response example (HTTP 409 Conflict)


```json
{
  "error": {
    "code": "CONFLICT",
    "message": "This event identifier and direction have already been processed with a different payload.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_31",
    "trace_id": "trace_ERR_31",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 72](G8_Platform_API_Schema_assets/workflow_page_72.png)


<!-- PDF page 73 -->

## STEP 5 | API CARD 31 · End all sessions after


Group 1 revokes a customer login


**Data Model table(s): session + session_sync_event**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Group 1 sends a trusted password-change or security-revocation notification for a customer. |
| Method + endpoint | POST /sessions/v1/parties/{party_id}/sessions/end-all |
| Owner microservice | session-service |
| Who can call | Trusted SERVICE caller: Group 1 only |
| Required headers | Trusted Group 1 service identity; Accept: application/json; X-Request-ID; Idempotency-Key. |
| Inputs and validation | Path party_id is Group 1 PARTY_ID. No customer bearer token and no browser-supplied party_id. Require reason G1_REVOKED or approved security reason. |
| Source tables / dependencies | session.session; session.session_sync_event; gateway.audit_event |
| Backend flow | Find ACTIVE sessions for party_id; set status=REVOKED, ended_at and end_reason=G1_REVOKED; create/record inbound sync event; invalidate sessions across all services. Never store passwords or raw tokens. |
| Frontend response | Force re-authentication through Group 1 on every device/session. |
| Safe to repeat? | Idempotent; repeated request ends no additional sessions. |
| Open points / owner decisions | Confirm whether Group 1 sends a webhook event instead of this party-level endpoint; confirm whether status REVOKED is used for password changes. |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/sessions/v1/parties/{party_id}/sessions/end-all",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "reason": "G1_REVOKED",
    "event_id": "g1-evt-55823"
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "party_id": "P-10012",
    "ended_sessions": 2,
    "status": "REVOKED",
    "end_reason": "G1_REVOKED",
    "ended_at": "2026-11-10T04:20:00Z"
  },
  "meta": {
    "request_id": "req_G8_037",
    "trace_id": "trace_G8_037",
    "timestamp": "2026-11-10T04:20:00Z",
    "simulated": true,
    "source": "session-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


<!-- PDF page 74 -->

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_32",
    "trace_id": "trace_ERR_32",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 74](G8_Platform_API_Schema_assets/workflow_page_74.png)


<!-- PDF page 75 -->

## STEP 6 | API CARD 32 · Gateway idempotency


processing (internal contract)


**Data Model table(s): idempotency_record**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Gateway intercepts a protected mutation/payment request carrying an Idempotency-Key. |
| Method + endpoint | INTERNAL MIDDLEWARE Gateway request interception (no customer-facing endpoint) |
| Owner microservice | api-gateway |
| Who can call | CUSTOMER, STAFF or SERVICE caller through Gateway |
| Required headers | Idempotency-Key required for protected create/change/payment operations; Authorization and X-Request-ID follow caller type. |
| Inputs and validation | Key is scoped by actor_id; fingerprint is generated from canonical request content. status IN_PROGRESS, COMPLETED, FAILED. Store result_ref ∈ and expiry; never expose fingerprint internals. |
| Source tables / dependencies | gateway.idempotency_record; access.actor; gateway.audit_event |
| Backend flow | Before execution, insert or find composite key (idempotency_key, actor_id). Same fingerprint + COMPLETED returns stored result_ref; IN_PROGRESS returns wait/status; different fingerprint under same key returns conflict. Mark final status and completed_at. |
| Frontend response | Disable duplicate submission; for UNKNOWN payment outcome, query owning service status rather than resubmit. |
| Safe to repeat? | A repeated request is safe only with same key and identical fingerprint. |
| Open points / owner decisions | Model description says money call, but workbook example 3 is a policy nominee change. Confirm whether table covers all protected mutations or money calls only; define expiry and retention. |


### Request example


```json
{
  "method": "INTERNAL MIDDLEWARE",
  "endpoint": "Gateway request interception (no customer-facing endpoint)",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "headers": {
      "Idempotency-Key": "idem-20261110-000101"
    },
    "body": {
      "example": "canonical request payload"
    }
  }
}
```


### Success response example (JSON)


<!-- PDF page 76 -->

```json
{
  "data": {
    "idempotency_key": "idem-20261110-000101",
    "actor_id": "a1000000-0000-4000-8000-000000000012",
    "request_fingerprint": "sha256:8d2a…",
    "status": "COMPLETED",
    "result_ref": "TXN-G2-5521",
    "created_at": "2026-11-10T04:34:00Z",
    "completed_at": "2026-11-10T04:34:01Z",
    "expires_at": "2026-11-11T04:34:00Z"
  },
  "meta": {
    "request_id": "req_G8_032",
    "trace_id": "trace_G8_032",
    "timestamp": "2026-11-10T04:34:01Z",
    "simulated": true,
    "source": "api-gateway"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_33",
    "trace_id": "trace_ERR_33",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 76](G8_Platform_API_Schema_assets/workflow_page_76.png)


<!-- PDF page 77 -->

## STEP 6 | API CARD 33 · Search Gateway audit


events


**Data Model table(s): audit_event**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Authorised reviewer searches actions by request, trace, session, caller service or time range. |
| Method + endpoint | GET /audit/v1/events |
| Owner microservice | audit-query-service (within Group 8; proposed service name) |
| Who can call | STAFF/REVIEWER with audit-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | At least one bounded filter: from/to, request_id, trace_id, session_id, caller_service_id or route_id. Require limit/cursor; restrict date range. |
| Source tables / dependencies | gateway.audit_event; session.session; registry.service_registry; gateway.route |
| Backend flow | Query immutable audit_event rows; join reference names only when authorised. Return event_id, request_id, trace_id, session_id, caller_service_id, route_id, action, outcome, occurred_at. Do not return credentials or personal payloads. |
| Frontend response | Show audit timeline and request trace; keep customer-safe messages separate from technical audit codes. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Data Model outcome enum has only CONFIRMED/FAILED/PENDING/UNKNOWN although audit_event covers every Gateway action. Confirm how successful reads and non-payment actions are represented. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/audit/v1/events",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "query": {
      "from": "2026-11-10T00:00:00Z",
      "to": "2026-11-10T23:59:59Z",
      "limit": 50
    }
  }
}
```


### Success response example (JSON)


<!-- PDF page 78 -->

```json
{
  "data": [
    {
      "event_id": "ae000000-0000-4000-8000-000000000001",
      "request_id": "req_G8_032",
      "trace_id": "trc-8f21a0",
      "session_id": "5e000000-0000-4000-8000-000000000004",
      "caller_service_id": null,
      "route_id": "f1000000-0000-4000-8000-000000000001",
      "action": "SESSION_START",
      "outcome": "CONFIRMED",
      "occurred_at": "2026-11-10T04:34:01Z"
    }
  ],
  "page": {
    "next_cursor": null,
    "limit": 50
  },
  "meta": {
    "request_id": "req_G8_033",
    "trace_id": "trace_G8_033",
    "timestamp": "2026-11-10T04:35:00Z",
    "simulated": true,
    "source": "audit-query-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_34",
    "trace_id": "trace_ERR_34",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


<!-- PDF page 79 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 79](G8_Platform_API_Schema_assets/workflow_page_79.png)


<!-- PDF page 80 -->

## STEP 7 | API CARD 34 · Submit an administrative


change request


**Data Model table(s): change_request**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | An authorised maker proposes a change to a controlled Platform record. |
| Method + endpoint | POST /internal/v1/change-requests |
| Owner microservice | change-management-service (Group 8; proposed service name) |
| Who can call | STAFF maker with change-request permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |
| Inputs and validation | Required target_table, target_id, new_value, reason, effective_from. old_value optional and should be captured by server. target_table must be one of the model’s enumerated tables. maker_id derived from caller; status starts PROPOSED. |
| Source tables / dependencies | config.change_request; access.actor; target master/controlled table; gateway.idempotency_record; gateway.audit_event |
| Backend flow | Validate target_table allowlist, target record, proposed field allowlist and reason. Capture old_value server-side; set maker_id, requested_at and PROPOSED. Do not apply new_value until approved and effective. |
| Frontend response | Show pending request with change_id, target, maker and effective date. Active configuration remains unchanged. |
| Safe to repeat? | Idempotent with same key/body. |
| Open points / owner decisions | Model target_table includes ACTOR_PERSONA and ADMIN_PROFILE, which are transactional, while Step 7 caption says approved changes update master data. Confirm intended scope and target_id encoding for composite-key PERSONA_PERMISSION. |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/internal/v1/change-requests",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "target_table": "SERVICE_REGISTRY",
    "target_id": "d1000000-0000-4000-8000-000000000002",
    "new_value": {"status": "LIVE"},
    "reason": "Approve service to go live (simulated)",
    "effective_from": "2026-11-11T00:00:00Z"
  }
}
```


### Success response example (JSON)


The target_table value must be one of the change_target enum values defined in Data Model v5.3. PERSONA_PERMISSION uses a composite key; target_id
encoding requires owner approval. Step 7 scope for ACTOR_PERSONA and ADMIN_PROFILE also requires confirmation.


<!-- PDF page 81 -->

```json
{
  "data": {
    "change_id": "cc000000-0000-4000-8000-000000000004",
    "target_table": "SERVICE_REGISTRY",
    "target_id": "d1000000-0000-4000-8000-000000000002",
    "old_value": {"status": "IN_BUILD"},
    "new_value": {"status": "LIVE"},
    "reason": "Approve service to go live (simulated)",
    "maker_id": "a1000000-0000-4000-8000-000000000902",
    "checker_id": null,
    "status": "PROPOSED",
    "requested_at": "2026-11-10T12:00:00Z",
    "decided_at": null,
    "effective_from": "2026-11-11T00:00:00Z"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The target table, target ID, or proposed field set is invalid.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_35",
    "trace_id": "trace_ERR_35",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 81](G8_Platform_API_Schema_assets/workflow_page_81.png)


<!-- PDF page 82 -->

## STEP 7 | API CARD 35 · Read a change request


**Data Model table(s): change_request**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | Maker, checker or authorised reviewer opens a change request and its current state. |
| Method + endpoint | GET /internal/v1/change-requests/{change_id} |
| Owner microservice | change-management-service (Group 8; proposed service name) |
| Who can call | STAFF maker/checker/reviewer with request-read permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Accept: application/json; X-Request-ID. |
| Inputs and validation | Path change_id UUID. Enforce visibility based on maker/checker/reviewerpermissions. |
| Source tables / dependencies | config.change_request; access.actor; target table read-only reference |
| Backend flow | Read change_request by change_id and return old/new values only to authorised staff. Mask any sensitive values in JSONB before display. |
| Frontend response | Show PROPOSED, APPROVED or REJECTED status, maker/checker, decision note and effective date. |
| Safe to repeat? | GET is safe to repeat. |
| Open points / owner decisions | Confirm JSONB field-level masking and whether makers can read checker decision notes before decision. |


### Request example


```json
{
  "method": "GET",
  "endpoint": "/internal/v1/change-requests/{change_id}",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "path": {
      "change_id": "cc000000-0000-4000-8000-000000000004"
    }
  }
}
```


### Success response example (JSON)


<!-- PDF page 83 -->

```json
{
  "data": {
    "change_id": "cc000000-0000-4000-8000-000000000004",
    "target_table": "SERVICE_REGISTRY",
    "target_id": "d1000000-0000-4000-8000-000000000002",
    "old_value": {"status": "IN_BUILD"},
    "new_value": {"status": "LIVE"},
    "reason": "Approve service to go live (simulated)",
    "maker_id": "a1000000-0000-4000-8000-000000000902",
    "checker_id": null,
    "decision_note": null,
    "status": "PROPOSED",
    "requested_at": "2026-11-10T12:00:00Z",
    "decided_at": null,
    "effective_from": "2026-11-11T00:00:00Z"
  },
  "meta": {
    "request_id": "req_G8_036",
    "trace_id": "trace_G8_036",
    "timestamp": "2026-11-10T12:01:00Z",
    "simulated": true,
    "source": "change-management-service"
  }
}
```


### Error response example (HTTP 400 Bad Request)


```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request could not be processed.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_36",
    "trace_id": "trace_ERR_36",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


<!-- PDF page 84 -->

### End-to-End API


![End-to-End API workflow diagram — original PDF page 84](G8_Platform_API_Schema_assets/workflow_page_84.png)


<!-- PDF page 85 -->

## STEP 7 | API CARD 36 · Approve or reject a


change request


**Data Model table(s): change_request**


| Card field | Detailed contract |
| --- | --- |
| Customer action / screen | A different authorised checker reviews a PROPOSED change and records a decision. |
| Method + endpoint | POST /internal/v1/change-requests/{change_id}/decision |
| Owner microservice | change-management-service (Group 8; proposed service name) |
| Who can call | STAFF checker with change-approve permission |
| Required headers | Authorization: Bearer &lt;staff-login-pass&gt;; Content-Type: application/json; Accept: application/json; X-Request-ID; Idempotency-Key. |
| Inputs and validation | Path change_id UUID. Body decision APPROVE, REJECT; optional ∈ decision_note. checker_id derived from caller and must differ from maker_id. |
| Source tables / dependencies | config.change_request; access.actor; target table; gateway.audit_event; gateway.idempotency_record |
| Backend flow | Lock request; require PROPOSED; enforce maker != checker. On REJECT set status REJECTED, checker_id, decision_note, decided_at. On APPROVE set APPROVED and apply new_value at effective_from using table-specific validation; update target record and audit. |
| Frontend response | Show final decision. If approved but effective_from is future, display scheduled state while old value remains active until effective time. |
| Safe to repeat? | Idempotent with same key and same decision. |
| Open points / owner decisions | Confirm whether approved changes apply immediately or at effective_from, and whether approval and activation are separate states. |


### Request example


```json
{
  "method": "POST",
  "endpoint": "/internal/v1/change-requests/{change_id}/decision",
  "headers": "As specified in the card; X-Request-ID is required.",
  "payload": {
    "decision": "APPROVE",
    "decision_note": "Reviewed and approved."
  }
}
```


### Success response example (JSON)


```json
{
  "data": {
    "change_id": "cc000000-0000-4000-8000-000000000004",
    "status": "APPROVED",
    "maker_id": "a1000000-0000-4000-8000-000000000902",
    "checker_id": "a1000000-0000-4000-8000-000000000901",
    "decision_note": "Reviewed and approved.",
    "requested_at": "2026-11-10T12:00:00Z",
    "decided_at": "2026-11-10T12:05:00Z",
    "effective_from": "2026-11-11T00:00:00Z"
  }
}
```


### Error response example (HTTP 403 Forbidden)


<!-- PDF page 86 -->

```json
{
  "error": {
    "code": "SAME_PERSON_NOT_ALLOWED",
    "message": "The maker cannot approve or reject their own change request.",
    "details": [],
    "retryable": false
  },
  "meta": {
    "request_id": "req_ERR_37",
    "trace_id": "trace_ERR_37",
    "timestamp": "2026-11-10T12:00:00Z",
    "simulated": true
  }
}
```


### End-to-End API


![End-to-End API workflow diagram — original PDF page 86](G8_Platform_API_Schema_assets/workflow_page_86.png)


<!-- PDF page 87 -->

## Appendix A - Data Model v5.3 Table-to-API Coverage


APPENDIX B - LEGACY MAPPINGS NOT CARRIED FORWARD
- user_service_access: not present in Data Model v5.3. Service visibility is derived through session, actor_persona, persona_permission, permission,
menu_item and service_registry.
- reference_code: not present in Data Model v5.3. No fictitious Group 8 reference-code table or endpoint is introduced.
- rule_change_log, platform_rule and policy_rule: not present in Data Model v5.3. Master-data changes use change_request and the Step 7 workflow.
- Payment-limit functionality is not part of the v5.3 Group 8 model. Payment limits and rail cut-offs are Group 4-owned; the former API has been removed.
- Legacy service_registry fields display_name, short_description, menu_label, menu_order, customer_facing, page_url, base_path and readiness are not
v5.3 columns. Use service_name, description, base_url, endpoint_path, health_url, contract_version and status. Menu presentation belongs in menu_item.
- Legacy role_id/access_id concepts are replaced by persona_id, permission_id, actor_persona_id and the composite persona_permission link.
- API paths in this revision are contract proposals unless a supplied source explicitly confirms them.


APPENDIX C - WIREFRAME SCOPE LIMITATIONS
Wireframes v2 defines Group 8 Home, All Services and Status only. Login and authentication are Group 1 responsibilities. The Platform does not invent
Group 1 login/password/OTP endpoints or a test-customer sign-in API. Group 1 GET /customers/v1/me is a Group 1-owned dependency, not a Group 8
API card.


APPENDIX D - AUTHORITATIVE SOURCES
- Group8_Platform_Data_Model_v5.3_Flow.pdf - authoritative 18-table inventory, seven-step sequence and relationship diagram.
- Group8_Platform_Data_Model_v5.3_Sample_Data.xlsx - authoritative table tabs, columns, types, PK/FK notes, required indicators, enums and simulated
examples.
- Group8_Platform_Wireframes_v2.pdf - incomplete reference for Home, All Services, Status and shared navigation.
- Existing Group 8 API Schema - retained as the formatting and API-card baseline, corrected where it conflicts with v5.3.


The former transaction-limit API 18 has been removed. The remaining 36 active API cards are renumbered consecutively from API 01 through API 36.


<!-- PDF page 88 -->

Relationship to this Data Model 
table


See the linked card for this 
table’s read/write operation and 
backend handling.


API-02
List all registered services
GET /registry/v1/services


See the linked card for this 
table’s read/write operation and 
backend handling.


API-03
Register a service
POST /registry/v1/services


See the linked card for this 
table’s read/write operation and 
backend handling.


API-04
Update a registered service
PATCH 
/registry/v1/services/{service_id}


See the linked card for this 
table’s read/write operation and 
backend handling.


API-05
List services available to the 
signed-in user
GET /registry/v1/me/services


See the linked card for this 
table’s read/write operation and 
backend handling.


API-06
Read service status
GET /registry/v1/status


### Step 1 — menu_item


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-07 | List menu items for a service | GET /registry/v1/menu-items? service_id={service_id} | See the linked card for this table’s read/write operation and backend handling. |
| API-08 | Create a menu item | POST /registry/v1/menu-items | See the linked card for this table’s read/write operation and backend handling. |
| API-09 | Update a menu item | PATCH /registry/v1/menu-items/{menu_item_id} | See the linked card for this table’s read/write operation and backend handling. |


### Step 2 — persona


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-10 | List personas | GET /access/v1/personas | See the linked card for this table’s read/write operation and backend handling. |
| API-11 | Create a persona | POST /access/v1/personas | See the linked card for this table’s read/write operation and backend handling. |


### Step 2 — permission


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-12 | List permissions | GET /access/v1/permissions | See the linked card for this table’s read/write operation and backend handling. |
| API-13 | Create a permission | POST /access/v1/permissions | See the linked card for this table’s read/write operation and backend handling. |


### Step 2 — persona_permission


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-14 | Read permissions assigned to a persona | GET /access/v1/personas/{persona_id}/permissions | See the linked card for this table’s read/write operation and backend handling. |
| API-15 | Assign a permission to a persona | PUT /access/v1/personas/{persona_id}/permissions/{permission_id} | See the linked card for this table’s read/write operation and backend handling. |


<!-- PDF page 89 -->

### Step 3 - route

### Step 3 route

|  | API card navigation |  | Operation |  | Endpoint |  |
| --- | --- | --- | --- | --- | --- | --- |
| API-16 |  | List configured API routes |  | GET /gateway/v1/routes |  | See the linked card for this table’s read/write operation and backend handling. |


### Step 3 — config_setting


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-17 | Read a configuration setting | GET /internal/v1/settings/{config_key} | See the linked card for this table’s read/write operation and backend handling. |


### Step 3 — bank_clock


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-18 | Read simulated bank date and time | GET /clock/v1/today | See the linked card for this table’s read/write operation and backend handling. |
| API-20 | Advance simulated bank date/time | POST /clock/v1/advance | See the linked card for this table’s read/write operation and backend handling. |


### Step 3 — business_calendar


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-19 | Read business calendar | GET /clock/v1/calendar | See the linked card for this table’s read/write operation and backend handling. |


### Step 4 — actor


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-22 | List Platform actors | GET /access/v1/actors | See the linked card for this table’s read/write operation and backend handling. |
| API-23 | List personas assigned to an actor | GET /access/v1/actors/{actor_id}/personas | See the linked card for this table’s read/write operation and backend handling. |
| API-24 | Request a persona assignment | POST /access/v1/actors/{actor_id}/personas | See the linked card for this table’s read/write operation and backend handling. |
| API-27 | Start a Platform session after Group 1 confirms login | POST /sessions/v1/sessions | See the linked card for this table’s read/write operation and backend handling. |


<!-- PDF page 90 -->

### Step 4 - actor persona (continued)

### Step 4 actor_persona (continued)

| API card navigation |  | Operation |  | Endpoint |  |
| --- | --- | --- | --- | --- | --- |
| API-23 | List personas assigned to an actor |  | GET /access/v1/actors/{actor_id}/personas |  | See the linked card for this table’s read/write operation and backend handling. |
| API-24 | Request a persona assignment |  | POST /access/v1/actors/{actor_id}/personas |  | See the linked card for this table’s read/write operation and backend handling. |
| API-25 | Approve or reject a persona assignment |  | POST /access/v1/actor-persona-assignments/{actor_persona_id}/ decision |  | See the linked card for this table’s read/write operation and backend handling. |
| API-27 | Start a Platform session after Group 1 confirms login |  | POST /sessions/v1/sessions |  | See the linked card for this table’s read/write operation and backend handling. |


### Step 4 — admin_profile


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-26 | Read the current admin profile | GET /access/v1/admin-profile/me | See the linked card for this table’s read/write operation and backend handling. |


### Step 5 — session


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-27 | Start a Platform session after Group 1 confirms login | POST /sessions/v1/sessions | See the linked card for this table’s read/write operation and backend handling. |
| API-28 | Check the current Platform session | GET /sessions/v1/sessions/current | See the linked card for this table’s read/write operation and backend handling. |
| API-29 | End the current Platform session (logout) | DELETE /sessions/v1/sessions/current | See the linked card for this table’s read/write operation and backend handling. |
| API-31 | End all sessions after Group 1 revokes a customer login | POST /sessions/v1/parties/{party_id}/se | See the linked card for this table’s read/write operation and |


### Step 5 — session_sync_event


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-29 | End the current Platform session (logout) | DELETE /sessions/v1/sessions/current | See the linked card for this table’s read/write operation and backend handling. |
| API-30 | Receive or retry a Group 1 session-sync event | POST /internal/v1/session-sync-events | See the linked card for this table’s read/write operation and |


### Step 6 — idempotency_record


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-20 | Advance simulated bank date/time | POST /clock/v1/advance | See the linked card for this table’s read/write operation and backend handling. |


<!-- PDF page 91 -->

|  | API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- | --- |
| API-24 |  | Request a persona assignment | /access/v1/actors/{actor_id}/personas | table’s read/write operation and backend handling. |
| API-25 |  | Approve or reject a persona assignment | POST /access/v1/actor-persona-assignments/{actor_persona_id}/ decision | See the linked card for this table’s read/write operation and backend handling. |
| API-31 |  | End all sessions after Group 1 revokes a customer login | POST /sessions/v1/parties/{party_id}/sessions/end-all | See the linked card for this table’s read/write operation and backend handling. |
| API-33 |  | Search Gateway audit events | GET /audit/v1/events | See the linked card for this table’s read/write operation and backend handling. |
| API-35 |  | Read a change request | GET /internal/v1/change-requests/{change_id} | See the linked card for this table’s read/write operation and backend handling. |


### Step 6 — audit_event


| API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- |
| API-03 | Register a service | POST /registry/v1/services | See the linked card for this table’s read/write operation and backend handling. |
| API-04 | Update a registered service | PATCH /registry/v1/services/{service_id} | See the linked card for this table’s read/write operation and backend handling. |
| API-08 | Create a menu item | POST /registry/v1/menu-items | See the linked card for this table’s read/write operation and backend handling. |
| API-09 | Update a menu item | PATCH /registry/v1/menu-items/{menu_item_id} | See the linked card for this table’s read/write operation and backend handling. |
| API-13 | Create a permission | POST /access/v1/permissions | See the linked card for this table’s read/write operation and backend handling. |
| API-20 | Advance simulated bank date/time | POST /clock/v1/advance | See the linked card for this table’s read/write operation and backend handling. |
| API-24 | Request a persona assignment | POST /access/v1/actors/{actor_id}/personas | See the linked card for this table’s read/write operation and backend handling. |
| API-25 | Approve or reject a persona assignment | POST /access/v1/actor-persona-assignments/{actor_persona_id}/ decision | See the linked card for this table’s read/write operation and backend handling. |
| API-29 | End the current Platform session (logout) | DELETE /sessions/v1/sessions/current | See the linked card for this table’s read/write operation and backend handling. |
| API-30 | Receive or retry a Group 1 session-sync event | POST /internal/v1/session-sync-events | See the linked card for this table’s read/write operation and backend handling. |
| API-31 | End all sessions after Group 1 revokes a customer login | POST /sessions/v1/parties/{party_id}/sessions/end-all | See the linked card for this table’s read/write operation and backend handling. |
| API-32 | Gateway idempotency processing (internal contract) | Gateway request interception (no customer-facing endpoint) | See the linked card for this table’s read/write operation and backend handling. |
| API-33 | Search Gateway audit events | GET /audit/v1/events | See the linked card for this table’s read/write operation and backend handling. |
| API-34 | Submit an administrative change request | POST /internal/v1/change-requests | See the linked card for this table’s read/write operation and backend handling. |
| API-35 | Read a change request | GET /internal/v1/change-requests/{change_id} | See the linked card for this table’s read/write operation and backend handling. |
| API-36 | Approve or reject a change request | POST /internal/v1/change-requests/{change_id}/decision | See the linked card for this table’s read/write operation and backend handling. |


<!-- PDF page 92 -->

### Step 7 - change_request Step 7 - change_request

|  | API card navigation | Operation | Endpoint | Relationship to this Data Model table |
| --- | --- | --- | --- | --- |
|  | API-34 | Submit an administrative change request | POST /internal/v1/change-requests | See the linked API card for the table read/write flow. |
|  |  |  | GET /i t l/ 1/ h t / | S th li k d API d f th t bl |
|  | API-35 | Read a change request | GET /internal/v1/change-requests/ {change_id} | See the linked API card for the table read/write flow. |
|  | API-36 | Approve or reject a change request | POST /internal/v1/change-requests/ {change id}/decision | See the linked API card for the table read/write flow |
| {change_id}/decisionread/write flow. |  |  | {change_id}/decision | read/write flow. |

