#  NoSQL Data Modeling Documentation

This section of the Crudzaso documentation covers the **NoSQL data modeling** decisions for the inspection results system. It describes the document structure used in the `inspection_results` collection and justifies the architectural choice of a hybrid SQL + NoSQL approach.

---

## 1. Document Structure

The `inspection_results` collection stores the results of inspections executed by field technicians. Each document represents a **complete, self-contained inspection**, allowing all related information to be retrieved in a single query without requiring multiple external lookups.

Each document is composed of root-level fields, embedded fields, and reference fields.

### Root-Level Fields

| Field | Type | Kind |
|---|---|---|
| `id` | `string` | Identifier |
| `company_id` | `string` | Reference → SQL |
| `inspection_id` | `string` | Reference → SQL |
| `form_version` | `string` | Value |
| `submitted_at` | `datetime` | Value |
| `submitted_by` | `string` | Reference → SQL |
| `status` | `string` | Value |
| `answers` | `array` | Embedded |
| `evidences` | `array` | Embedded |
| `metadata` | `object` | Embedded |

> Fields marked as **Reference → SQL** point to entities managed in the relational database. Fields marked as **Embedded** are detailed in section 2. See section 3 for reference field justification.

### Document Example

```json
{
  "id": "result_67a3f2e1b4c9d8e5a1234567",
  "company_id": "18237498",
  "inspection_id": "INS-2024-001",
  "form_version": "v2.1",
  "submitted_at": "2024-11-15T14:32:00Z",
  "submitted_by": "user_123",
  "status": "completed",
  "answers": [
    {
      "question_id": "q1",
      "question_text": "Is the equipment operational?",
      "value": true
    },
    {
      "question_id": "q2",
      "question_text": "Engine temperature (°C)",
      "value": 78.5,
      "unit": "celsius"
    },
    {
      "question_id": "q3",
      "question_text": "Additional observations",
      "value": "A slight vibration was detected in the left bearing"
    },
    {
      "question_id": "q4",
      "question_text": "Overall condition",
      "value": "good",
      "options": ["excellent", "good", "fair", "poor"]
    }
  ],
  "evidences": [
    {
      "evidence_id": "ev_001",
      "question_id": "q2",
      "type": "image",
      "url": "https://storage.crudzaso.com/evidence/img_20241115_143200.jpg",
      "filename": "motor_thermography.jpg",
      "uploaded_at": "2024-11-15T14:32:15Z"
    },
    {
      "evidence_id": "ev_002",
      "question_id": "q3",
      "type": "audio",
      "url": "https://storage.crudzaso.com/evidence/audio_20241115_143245.m4a",
      "filename": "bearing_noise.m4a",
      "uploaded_at": "2024-11-15T14:32:45Z"
    }
  ],
  "metadata": {
    "device_info": "iPhone 14 Pro - iOS 17.1",
    "location": {
      "latitude": 6.2442,
      "longitude": -75.5812,
      "accuracy": 10
    },
    "duration_minutes": 23,
    "synced_at": "2024-11-15T14:35:00Z"
  }
}
```

---

## 2. What Gets Embedded?

The following fields are modeled using **embedding** because their data is tightly coupled to the inspection document and has no meaningful existence outside of it.

**`answers`** – An array containing all responses recorded during the inspection. Each entry includes the question ID, question text, and the recorded value. Depending on the response type, it may also include measurement units or selection options. Answers are embedded because they are integral to the inspection and are always retrieved alongside it.

**`evidences`** – An array of files associated with the inspection (images, audio, etc.). Each evidence entry contains its ID, file type, external storage URL, filename, and upload timestamp. Evidences are embedded because they belong exclusively to the inspection instance and are never consulted independently.

**`metadata`** – Technical and contextual information about the inspection execution, including device info, geographic coordinates, duration in minutes, and sync timestamp. This data is specific to a single execution and does not constitute an independent entity, so it is embedded directly in the document.

---

## 3. What Gets Referenced?

The following fields are **not embedded**. Instead, they store only an identifier that points to an entity managed in the relational (SQL) database. This avoids data duplication and keeps the source of truth for stable, structured entities in the appropriate system.

**`company_id`** – References the `companies` table in SQL. Detailed company information (name, sector, subscription plan, registered users) is managed relationally. Only the identifier is stored in the document to establish ownership without duplicating data.

**`inspection_id`** – References the inspection assignment record in SQL. This is the **bridge between both databases**, linking the result to its formal assignment, inspection type, and form definition. It allows the document to be contextualized without embedding administrative data that belongs in the relational layer.

**`submitted_by`** – References the `users` table in SQL. User information such as name, role, and permissions is centrally managed in the relational database. Only the user identifier is stored here to attribute authorship of the inspection result.

---

## 4. Why This Data Does Not Belong in SQL

Inspection results are not suitable for a relational model due to their **structural variability**. Each inspection type may contain different questions, different response formats (booleans, numeric values, free text, selection lists), and a variable number of associated files. There is no single fixed schema that could represent all possible inspection forms without introducing significant complexity.

Attempting to store this data in SQL would require either a rigid table with many nullable columns, or an overly generic key-value table that sacrifices readability and queryability. Additionally, as forms evolve over time — adding, removing, or modifying questions — the relational schema would require constant migrations, increasing maintenance overhead and the risk of breaking existing records.

---

## 5. Why This Data Belongs in NoSQL

A document-oriented database is the right fit for inspection results because it naturally accommodates **flexible, evolving structures**. Each inspection result can be stored as a self-contained document with its own shape, without requiring a predefined schema shared across all records.

The `form_version` field allows historical records to preserve the exact structure used at execution time, even as forms change in the future. Embedding answers, evidences, and metadata within the same document enables efficient full-document retrieval in a single query, which matches the most common access pattern: loading a complete inspection result for review or reporting.

---

## 6. SQL vs. NoSQL Comparison

### Advantages

| Dimension | Relational Model (SQL) | Document Model (NoSQL) |
|---|---|---|
| Data consistency | ACID transactional guarantees | Eventual consistency (configurable) |
| Schema | Fixed, enforced structure | Flexible, schema-free |
| Referential integrity | Guaranteed via foreign keys | Manual, application-level |
| Query complexity | Supports complex JOINs | Optimized for document retrieval |
| Scalability | Vertical scaling | Horizontal scaling |
| Best suited for | Stable, structured entities | Dynamic, variable data |

### Risks

**SQL risks:** Rigidity when the data structure changes frequently, requiring schema migrations (`ALTER TABLE`). Complexity increases as data becomes more variable, and tight coupling between tables grows over time.

**NoSQL risks:** No automatic referential integrity, which introduces risk of inconsistency if cross-database references are not carefully managed. Data duplication is possible, and structural validation is less strict unless additional application-level controls are implemented.

### Business Coherence

The Crudzaso system combines structured, stable information with dynamic, variable data. Administrative entities — companies, users, roles, and assignments — require high consistency and relational control, making SQL the natural fit. Inspection results, on the other hand, exhibit structural variability depending on the form type, making NoSQL the more appropriate storage layer without generating unnecessary complexity.

The hybrid architecture leverages the strengths of both models: **SQL for structured business data management**, and **NoSQL for flexible storage of dynamic inspection data**. This decision maintains coherence with the system's functional requirements, optimizing both integrity and adaptability.

---

## 7. Architectural Justification

The decision to use a document-oriented database for inspection results responds directly to the dynamic and evolving nature of the data captured in the field. Inspections do not follow a fixed, uniform structure — each type may contain different questions, varied response formats, and multiple associated files. Forms may also evolve over time, adding or modifying fields without notice.

For these reasons, the NoSQL database is used exclusively to store dynamic inspection results, while structured and stable data (companies, users, roles, and assignments) is maintained in the relational database. This separation of concerns is the foundation of Crudzaso's **hybrid architecture**, which allows each type of data to be stored in the system best suited for its characteristics.

The `inspection_id` field serves as the explicit connection point between both systems, enabling a result document to be associated with its relational context without duplicating data across layers.

---