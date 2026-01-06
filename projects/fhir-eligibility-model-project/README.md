# Insurance Eligibility Data Model (HL7 FHIR)

## Project Overview

In healthcare, verifying if a patient has active insurance and what their benefits cover (deductibles, copays) is a critical step before care is provided. This project models the "Transaction Triple" using the **HL7® FHIR® R4** standard to facilitate real-time eligibility checks between a Provider (EHR) and an Insurer (Payer).

The following Entity Relationship Diagram (ERD) visualizes the logical flow of an insurance eligibility check. It maps how a patient’s static coverage information transitions into a dynamic request-and-response transaction. This model ensures that all participants (Patient, Provider, and Insurer) are synchronized through linked FHIR resources.

## Data Architecture (ERD)

```mermaid
erDiagram
    %% Core Entities and Relationships
    PATIENT ||--o{ COVERAGE : "is beneficiary of"
    ORGANIZATION ||--o{ COVERAGE : "is payor for"
    PATIENT ||--o{ COVERAGE_ELIGIBILITY_REQUEST : "is subject of"
    ORGANIZATION ||--o{ COVERAGE_ELIGIBILITY_REQUEST : "is recipient (insurer) of"
    
    COVERAGE ||--o{ COVERAGE_ELIGIBILITY_REQUEST : "is being verified"
    COVERAGE_ELIGIBILITY_REQUEST ||--o| COVERAGE_ELIGIBILITY_RESPONSE : "expects/receives"
    
    %% Entity Definitions with Summary Fields
    COVERAGE {
        string id PK "Logical ID"
        identifier identifier "Business ID (Policy/Group)"
        code status "active | cancelled | draft"
        CodeableConcept type "HMO, PPO, etc."
        Reference subscriber "Link to Patient/RelatedPerson"
        string subscriberId "Member ID on card"
        Reference payor "Link to Organization"
        Period period "Coverage start/end"
        integer order "Coordination of Benefits priority"
    }

    COVERAGE_ELIGIBILITY_REQUEST {
        string id PK
        identifier identifier "Inquiry ID"
        code status "active | cancelled | draft"
        code purpose "auth-requirements | benefits | discovery | validation"
        Reference patient FK "Link to Patient"
        dateTime created "Request timestamp"
        Reference provider "Requesting Practitioner/Org"
        Reference insurer FK "Target Payor"
        date serviced "Planned date of service"
        boolean insurance_focal "Is this the primary coverage?"
    }

    COVERAGE_ELIGIBILITY_RESPONSE {
        string id PK
        identifier identifier "Response ID"
        code status "active | cancelled | draft"
        code outcome "queued | complete | error | partial"
        string disposition "Human-readable result message"
        dateTime created "Response timestamp"
        Reference request FK "Link back to Request"
        Reference insurer "The responding Payor"
        string preAuthRef "Auth number (if requested)"
    }

    %% Showing the complex Benefit Item breakdown
    COVERAGE_ELIGIBILITY_RESPONSE ||--o{ RESPONSE_ITEM : "contains"
    RESPONSE_ITEM {
        CodeableConcept category "Dental, Vision, etc."
        CodeableConcept productOrService "CPT/HCPCS code"
        boolean excluded "Is service specifically excluded?"
        string benefit_summary "Deductible, Copay, etc."
    }
```

## FHIR Resources Used

* **[Coverage](https://www.hl7.org/fhir/R4/coverage.html):** Represents the "Insurance Card." It contains the member ID, the payer (e.g., Aetna, Medicare), and the plan type.
* **[CoverageEligibilityRequest](https://www.hl7.org/fhir/R4/coverageeligibilityrequest.html):** The digital "question" sent by the provider to ask if specific services are covered on a specific date.
* **[CoverageEligibilityResponse](https://www.google.com/search?q=https://www.hl7.org/fhir/R4/coverageeligibilityresponse.html):** The digital "answer" containing the outcome, authorization requirements, and detailed benefit items (copays/deductibles).

## Implementation Examples

You can find valid FHIR JSON samples for these resources in the `/samples` folder:

* [Coverage Sample](./samples/coverage.json)
* [Eligibility Request](./samples/request.json)
* [Eligibility Response](./samples/response.json)

## Key Learning Outcomes

* **Interoperability:** Mapping complex healthcare workflows to the international FHIR R4 standard.
* **Cardinality:** Managing 1:N relationships between a single policy (`Coverage`) and multiple verification attempts (`Requests`).
* **Data Modeling:** Identifying "Summary" fields (Σ) essential for lightweight, performant API transactions.
