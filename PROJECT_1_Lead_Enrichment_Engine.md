# Project 1: Advanced Lead Qualification & Enrichment Engine

**Goal:** Build an automated system that enriches newly created Leads with external data via an API callout and assigns them to specific queues based on configurable rules.

**Core Technologies:** Apex Triggers, Apex Callouts (REST), Custom Metadata Types, Apex Unit Testing (Mocks), Salesforce Queues, SFDX.

---

## Project Backlog (Sequential Tasks)

1.  **Setup & Initialization:**
    *   [ ] Create SFDX Project (`sf project generate --name LeadEnrichmentEngine`).
    *   [ ] Authorize Dev Hub (`sf org login web --set-default-dev-hub --alias myhub`).
    *   [ ] Create Scratch Org (`sf org create scratch --set-default --definition-file config/project-scratch-def.json --alias LeadEnrich`).
    *   [ ] Push initial empty structure (`sf project deploy start`).
    *   [ ] Open project in VS Code.

2.  **Configuration (Custom Metadata):**
    *   [ ] Define Custom Metadata Type `Lead_Routing_Rule__mdt`.
        *   Fields: `Label`, `DeveloperName`, `Criteria_Field_API_Name__c` (Text), `Criteria_Value__c` (Text), `Target_Queue_DeveloperName__c` (Text), `Enrichment_API_Endpoint__c` (URL), `Enrichment_API_Key__c` (Text - Protected), `Is_Active__c` (Checkbox).
    *   [ ] Create 2-3 sample Queues in the scratch org (e.g., 'High Priority Leads', 'Industry A Leads').
    *   [ ] Create sample `Lead_Routing_Rule__mdt` records referencing the queues and potential criteria (e.g., Criteria Field: `Industry`, Value: `Technology`). Define a record for API details.
    *   [ ] Add Custom Fields to Lead object if needed for enrichment data (e.g., `Enriched_Industry__c`, `Enriched_Company_Size__c`).
    *   [ ] Retrieve metadata (`sf project retrieve start`) or track source manually.

3.  **Core Logic - Enrichment Service (Apex):**
    *   [ ] Create Apex class `LeadEnrichmentService`.
    *   [ ] Add method `enrichLeads(List<Lead> leads)` or similar.
    *   [ ] Inside the method:
        *   [ ] Query active `Lead_Routing_Rule__mdt` record for API details.
        *   [ ] Construct HTTP Request (GET/POST depending on API) to the enrichment API (use a simple public API like `https://api.zippopotam.us/us/90210` for testing the *pattern* if a real enrichment API isn't readily available/free).
        *   [ ] Send the request using `Http` class.
        *   [ ] Handle the `HttpResponse`.
        *   [ ] Parse the JSON response.
        *   [ ] Update the corresponding Lead records in the input list with enriched data (e.g., set `Enriched_Industry__c`). *Do not perform DML here.*
        *   [ ] Implement basic error handling (try-catch, `System.debug`).

4.  **Core Logic - Routing Service (Apex):**
    *   [ ] Create Apex class `LeadRoutingService`.
    *   [ ] Add method `routeLeads(List<Lead> leads)`.
    *   [ ] Inside the method:
        *   [ ] Query active `Lead_Routing_Rule__mdt` records for *routing* rules.
        *   [ ] Query existing Queues to get their IDs by DeveloperName.
        *   [ ] Iterate through the input `leads`.
        *   [ ] For each lead, evaluate the routing rules against its data (including potentially enriched data).
        *   [ ] If a rule matches, update the `OwnerId` of the Lead in the list to the corresponding Queue ID. *Do not perform DML here.*
        *   [ ] Implement logic for rule priority or first match.

5.  **Trigger Implementation:**
    *   [ ] Create Apex Trigger `LeadTrigger` on `Lead` (after insert).
    *   [ ] Create Apex Class `LeadTriggerHandler`.
    *   [ ] Implement trigger handler pattern (static method in handler called by trigger).
    *   [ ] In the handler method (`handleAfterInsert(List<Lead> newLeads)`):
        *   [ ] Instantiate `LeadEnrichmentService` and call `enrichLeads(newLeads)`. (Decide Sync vs Async - start with Sync for simplicity).
        *   [ ] Instantiate `LeadRoutingService` and call `routeLeads(newLeads)`.
        *   [ ] Perform necessary DML update on the modified `newLeads` list *only if changes were made* by routing (enrichment modifies list in memory, routing assigns owner).

6.  **Testing:**
    *   [ ] Add API endpoint to Remote Site Settings in scratch org.
    *   [ ] Create Apex Test Class `LeadEnrichmentService_Test`.
        *   [ ] Implement `HttpCalloutMock` to simulate API responses (success and failure).
        *   [ ] Write test methods to verify lead enrichment logic.
    *   [ ] Create Apex Test Class `LeadRoutingService_Test`.
        *   [ ] Create test Queues and `Lead_Routing_Rule__mdt` records in test setup (`@TestSetup`).
        *   [ ] Write test methods to verify routing logic based on different criteria.
    *   [ ] Create Apex Test Class `LeadTriggerHandler_Test`.
        *   [ ] Write test methods to simulate Lead inserts.
        *   [ ] Use `Test.startTest/stopTest`.
        *   [ ] Verify enrichment *and* routing occur correctly via asserts (SOQL queries). Aim for >85% coverage overall.
    *   [ ] Perform manual test: Create a Lead, check if custom fields populate (if enrichment worked) and if the Owner changes to the expected Queue.

7.  **Documentation & Cleanup:**
    *   [ ] Create/Update `README.md` in the project root.
        *   [ ] Explain project purpose, features.
        *   [ ] Detail setup steps (Metadata deployment, Remote Site Settings).
        *   [ ] Briefly explain design choices (Sync vs Async callout, Trigger Handler Pattern).
    *   [ ] Ensure all code is formatted correctly.
    *   [ ] Push final code and metadata to scratch org (`sf project deploy start`).
    *   [ ] Commit all changes to Git (`git add .`, `git commit -m "feat: Implement lead enrichment and routing"`).
    *   [ ] Push to GitHub repository.
