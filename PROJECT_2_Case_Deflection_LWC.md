# Project 2: Case Deflection with External Knowledge Base Search

**Goal:** Create an LWC on the Case page allowing agents to search an external source (like Stack Exchange API) based on Case details and view results directly in Salesforce.

**Core Technologies:** Lightning Web Components (LWC), Apex Controllers (@AuraEnabled), Apex Callouts (REST), Unit Testing (Mocks), SFDX.

---

## Project Backlog (Sequential Tasks)

1.  **Setup & Initialization:**
    *   [ ] Create SFDX Project (`sf project generate --name CaseDeflectionLWC`).
    *   [ ] Authorize Dev Hub.
    *   [ ] Create Scratch Org (`sf org create scratch --set-default --definition-file config/project-scratch-def.json --alias CaseDeflect`).
    *   [ ] Push initial empty structure.
    *   [ ] Open project in VS Code.

2.  **Apex Controller:**
    *   [ ] Create Apex Class `ExternalKnowledgeSearchController`.
    *   [ ] Add method `searchArticles(String searchTerm)` annotated with `@AuraEnabled(cacheable=true)`.
    *   [ ] Inside the method:
        *   [ ] Construct the REST API callout URL (e.g., Stack Exchange Search API).
        *   [ ] Create `HttpRequest`, set method to GET, set endpoint URL.
        *   [ ] Send request using `Http` class.
        *   [ ] Handle `HttpResponse`.
        *   [ ] Parse the JSON response. Focus on extracting relevant fields (e.g., question title, link, score/relevance).
        *   [ ] *Optional:* Define a simple inner Apex wrapper class (e.g., `SearchResultWrapper`) with `@AuraEnabled` properties to structure the data cleanly for the LWC.
        *   [ ] Return the list of results (e.g., `List<SearchResultWrapper>`).
        *   [ ] Implement try-catch error handling. Return an empty list or throw an `AuraHandledException` on error.

3.  **LWC Development - Structure & Style:**
    *   [ ] Create LWC component (`sf lightning generate component --type lwc --name externalKnowledgeSearch --output-dir force-app/main/default/lwc`).
    *   [ ] Edit `externalKnowledgeSearch.js-meta.xml`:
        *   [ ] Set `isExposed` to `true`.
        *   [ ] Define targets (e.g., `lightning__RecordPage`).
        *   [ ] Define target properties if needed (like `recordId`).
    *   [ ] Edit `externalKnowledgeSearch.html`:
        *   [ ] Add a `lightning-card` for structure (title: "External Knowledge Search").
        *   [ ] Add a `lightning-button` ("Search Knowledge").
        *   [ ] Add an area to display results (e.g., using `template if:true={results}` and `template for:each={results}`).
        *   [ ] Use `lightning-layout` and `lightning-layout-item` for basic formatting of results (e.g., display title and link).
        *   [ ] Add a loading spinner (`lightning-spinner`).
        *   [ ] Add an area to display errors.

4.  **LWC Development - Logic (JS):**
    *   [ ] Edit `externalKnowledgeSearch.js`:
        *   [ ] Import the Apex controller method (`import searchArticles from '@salesforce/apex/ExternalKnowledgeSearchController.searchArticles';`).
        *   [ ] Import `getRecord`, `getFieldValue` from `lightning/uiRecordApi`.
        *   [ ] Import `ShowToastEvent` for error notifications.
        *   [ ] Import `CASE_SUBJECT_FIELD`, `CASE_DESCRIPTION_FIELD` from `@salesforce/schema/Case`.
        *   [ ] Define `@api recordId`.
        *   [ ] Define tracked properties for `results`, `isLoading`, `error`.
        *   [ ] Use the `@wire(getRecord, ...)` adapter to fetch the Case Subject/Description based on `recordId`.
        *   [ ] Create a handler function for the button click (`handleSearch`).
        *   [ ] Inside `handleSearch`:
            *   Set `isLoading = true`, clear previous `results` and `error`.
            *   Get search terms from the wired Case data (e.g., Subject).
            *   Call the imported Apex method (`searchArticles({ searchTerm: ... })`).
            *   Use `.then(result => { ... })` to handle success: update `results`, set `isLoading = false`.
            *   Use `.catch(error => { ... })` to handle errors: update `error`, set `isLoading = false`, show a toast message.

5.  **Integration & Configuration:**
    *   [ ] Add the target API endpoint (e.g., `api.stackexchange.com`) to Remote Site Settings in the scratch org.
    *   [ ] Push LWC and Apex code to scratch org (`sf project deploy start`).
    *   [ ] Open a Case record page in the scratch org.
    *   [ ] Edit the Page (Lightning App Builder).
    *   [ ] Drag the `externalKnowledgeSearch` LWC onto the page layout (e.g., sidebar).
    *   [ ] Save and activate the page.

6.  **Testing:**
    *   [ ] Create Apex Test Class `ExternalKnowledgeSearchController_Test`.
        *   [ ] Implement `HttpCalloutMock` for the Stack Exchange API (success and failure scenarios).
        *   [ ] Write test methods to call `searchArticles` and assert the returned wrapper objects are correct based on the mock response. Test error handling.
    *   [ ] Perform manual testing:
        *   [ ] Navigate to the Case record page. Verify the LWC loads.
        *   [ ] Ensure the "Search Knowledge" button is present.
        *   [ ] Click the button. Verify the loading spinner appears.
        *   [ ] Verify results are displayed correctly (or an error message if applicable).
        *   [ ] Test with different Case subjects.

7.  **Documentation & Cleanup:**
    *   [ ] Create/Update `README.md`.
        *   [ ] Explain project purpose, features (LWC searching external API).
        *   [ ] Detail setup steps (Deployment, Remote Site Settings, Adding LWC to Page Layout).
        *   [ ] Include screenshots if helpful.
    *   [ ] Ensure all code is formatted correctly.
    *   [ ] Push final code and metadata to scratch org (`sf project deploy start`).
    *   [ ] Commit all changes to Git (`git add .`, `git commit -m "feat: Implement case deflection LWC"`).
    *   [ ] Push to GitHub repository.
