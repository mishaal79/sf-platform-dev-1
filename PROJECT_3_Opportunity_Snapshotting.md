# Project 3: Opportunity Stage-Based Project Snapshotting

**Goal:** Automatically create a related "Project Snapshot" custom record when an Opportunity reaches a specific stage (e.g., "Closed Won"), capturing key details and optionally notifying an external system.

**Core Technologies:** Custom Objects, Flows (Record-Triggered), Apex Invocable Actions, SOQL (Relationships), Asynchronous Apex (@future Callout - Optional), Unit Testing, SFDX.

---

## Project Backlog (Sequential Tasks)

1.  **Setup & Initialization:**
    *   [ ] Create SFDX Project (`sf project generate --name OpportunitySnapshot`).
    *   [ ] Authorize Dev Hub.
    *   [ ] Create Scratch Org (`sf org create scratch --set-default --definition-file config/project-scratch-def.json --alias OppSnapshot`).
    *   [ ] Push initial empty structure.
    *   [ ] Open project in VS Code.

2.  **Data Model (Custom Object):**
    *   [ ] Define Custom Object `Project_Snapshot__c`.
    *   [ ] Add Fields:
        *   `Opportunity__c` (Master-Detail or Lookup to Opportunity)
        *   `Account__c` (Lookup to Account)
        *   `Primary_Contact__c` (Lookup to Contact) - *Optional*
        *   `Snapshot_Amount__c` (Currency)
        *   `Snapshot_Date__c` (Date)
        *   `Line_Item_Details__c` (Long Text Area) - *Optional*
        *   `External_Project_ID__c` (Text) - *Optional, for callout response*
    *   [ ] Adjust field-level security/permissions as needed (defaults usually okay for scratch org).
    *   [ ] Add the related list for `Project Snapshots` to the Opportunity page layout.
    *   [ ] Retrieve metadata (`sf project retrieve start`) or track source manually.

3.  **Core Logic - Invocable Apex:**
    *   [ ] Create Apex Class `OpportunitySnapshotAction`.
    *   [ ] Create an inner wrapper class `SnapshotRequest` with `@InvocableVariable` annotation for input (e.g., `public ID oppId;`).
    *   [ ] Create method `createSnapshots(List<SnapshotRequest> requests)` annotated with `@InvocableMethod(label='Create Project Snapshot' description='Creates a Project Snapshot record from an Opportunity.')`.
    *   [ ] Inside the method:
        *   [ ] Extract Opportunity IDs from the input `requests`.
        *   [ ] Perform a SOQL query on Opportunity WHERE Id IN :oppIds, fetching necessary fields (`AccountId`, `Amount`, `PrimaryContactId__c` [if used], `(SELECT Product2.Name, Quantity, TotalPrice FROM OpportunityLineItems)`).
        *   [ ] Create a `List<Project_Snapshot__c>` to hold new snapshots.
        *   [ ] Iterate through the queried Opportunities.
        *   [ ] For each Opportunity, create a new `Project_Snapshot__c` record.
        *   [ ] Map Opportunity fields to Snapshot fields (e.g., `opp.AccountId` -> `snapshot.Account__c`).
        *   [ ] *Optional:* Format Line Item details into `Line_Item_Details__c`.
        *   [ ] Set `Snapshot_Date__c` to `System.today()`.
        *   [ ] Add the new snapshot record to the list.
        *   [ ] Perform `insert snapshotList;` after the loop.
        *   [ ] *Optional Callout Trigger:* Call a separate future method here, passing necessary data (e.g., Snapshot IDs and relevant details) to initiate the external callout. `ExternalProjectSystemService.initiateProject(snapshotData);`

4.  **Core Logic - Async Callout (Optional):**
    *   [ ] Create Apex Class `ExternalProjectSystemService`.
    *   [ ] Add method `initiateProject(List<ID> snapshotIds)` annotated with `@future(callout=true)`.
        *   *Alternative:* Pass structured data instead of just IDs if needed.
    *   [ ] Inside the method:
        *   [ ] Query `Project_Snapshot__c` records WHERE Id IN :snapshotIds, fetching data needed for the external system.
        *   [ ] Construct the JSON payload for the POST request.
        *   [ ] Create `HttpRequest`, set method POST, set endpoint URL (use a mock endpoint like RequestBin/Pipedream), set header `Content-Type: application/json`.
        *   [ ] Set the request body (`req.setBody(...)`).
        *   [ ] Send request using `Http` class.
        *   [ ] Handle `HttpResponse`.
        *   [ ] *Optional:* Parse response (e.g., an external project ID). If received, update the corresponding `Project_Snapshot__c` record(s) with the `External_Project_ID__c`. (Requires querying again or passing more data to future method).
        *   [ ] Add error handling/logging.

5.  **Automation (Flow):**
    *   [ ] Create a new Flow: Record-Triggered Flow.
    *   [ ] Object: Opportunity.
    *   [ ] Trigger: A record is updated.
    *   [ ] Condition Requirements: Formula Evaluates to True.
        *   Formula: `AND( ISCHANGED({!$Record.StageName}), ISPICKVAL({!$Record.StageName}, 'Closed Won') )` (Adjust 'Closed Won' as needed).
    *   [ ] Optimize the Flow for: Actions and Related Records.
    *   [ ] Add Action element: Search for and select your Apex Action ("Create Project Snapshot").
    *   [ ] Set Input Values: Create an SObject Variable for `SnapshotRequest` (or use individual inputs if your Invocable method expects simple types directly mapped), set the `oppId` input variable to `{!$Record.Id}`.
    *   [ ] Save and Activate the Flow.

6.  **Configuration (Optional Callout):**
    *   [ ] If implementing the callout, add the mock endpoint URL to Remote Site Settings.

7.  **Testing:**
    *   [ ] Create Apex Test Class `OpportunitySnapshotAction_Test`.
        *   [ ] Create test Account, Contact, Opportunity, and Opportunity Line Items in test setup (`@TestSetup`).
        *   [ ] Write test method(s) to call the invocable action directly (simulating Flow).
        *   [ ] Use `Test.startTest/stopTest`.
        *   [ ] Assert that `Project_Snapshot__c` record(s) are created correctly with mapped data via SOQL query.
    *   [ ] Create Apex Test Class `ExternalProjectSystemService_Test` (if callout implemented).
        *   [ ] Implement `HttpCalloutMock` to simulate the external system's response.
        *   [ ] Write test method(s) calling the `@future` method.
        *   [ ] Use `Test.startTest/stopTest` to ensure async execution.
        *   [ ] Assert the callout was made (using mock verification) and/or that the `Project_Snapshot__c` was updated with the external ID (if applicable).
    *   [ ] Perform manual testing:
        *   [ ] Create an Opportunity with Line Items.
        *   [ ] Update the Opportunity Stage to "Closed Won".
        *   [ ] Verify a `Project_Snapshot__c` record is created under the Opportunity's related list. Check field values.
        *   [ ] If callout implemented, check your mock endpoint (RequestBin/Pipedream) to see if the request arrived.

8.  **Documentation & Cleanup:**
    *   [ ] Create/Update `README.md`.
        *   [ ] Explain project purpose (snapshotting Opps), features (Flow + Apex).
        *   [ ] Detail setup steps (Deployment, Flow activation, Remote Site Settings if needed).
        *   [ ] Explain design (Why Invocable Apex? Why optional future callout?).
    *   [ ] Ensure all code is formatted correctly.
    *   [ ] Push final code and metadata to scratch org (`sf project deploy start`).
    *   [ ] Commit all changes to Git (`git add .`, `git commit -m "feat: Implement opportunity snapshotting"`).
    *   [ ] Push to GitHub repository.
