---
id: interpretacion-negocio
title: Business Interpretation
sidebar_position: 2
---

# Business Interpretation

## What is InspectaPro from a business perspective?

InspectaPro is a SaaS platform designed for companies that perform operational field inspections, such as technical maintenance, internal audits, quality control, inventory verification, or industrial safety.

The value of the system is not only to store data, but to **standardize inspection processes**, ensure **historical traceability**, and allow **verifiable evidence** of what happens in the field.

Each client company can:

- Define its own inspection types according to its operations.
- Register its technical and administrative staff.
- Assign inspections to specific technicians.
- Record results with evidence.
- Consult historical records for auditing and internal control.

The system is conceived as a **multi-company platform**, where each company has its own users, configurations, and data, maintaining logical isolation between clients.

---

## System Actors

### Company Administrator

Represents the internal manager of the client company. Their functions are:

- Create and manage users.
- Define roles and permissions.
- Create and modify inspection types.
- Create inspections.
- Assign inspections to technicians.
- Consult reports and historical records.

### Technician

Represents the personnel who execute inspections in the field. Their functions are:

- View assigned inspections.
- Execute inspections.
- Register answers.
- Attach evidence.
- Close inspections.

### Company (as an organizational entity)

Represents the client organization. It does not act directly, but:

- Owns users.
- Owns inspections.
- Owns inspection types.
- Has an active subscription.

---

## Main Business Rules

| # | Rule |
|---|------|
| 1 | Each company manages only its own users, inspections, and inspection types. |
| 2 | Only administrators can create inspection types. |
| 3 | Only administrators can create inspections. |
| 4 | Each inspection must be associated with an inspection type. |
| 5 | Each inspection must be assigned to a technician before execution. |
| 6 | A technician can only execute inspections that have been assigned to them. |
| 7 | An inspection cannot be closed if it has no recorded answers. |
| 8 | Inspection results cannot be modified after closure. |
| 9 | Each result must preserve the version of the form used. |
| 10 | The inspection history must be accessible for auditing purposes. |

---

## Use Cases

### Administrator Use Cases

- Manage users.
- Manage roles.
- Create inspection type.
- Modify inspection type.
- Create inspection.
- Assign inspection.
- View inspection history.
- View results.

### Technician Use Cases

- View assigned inspections.
- Execute inspection.
- Record answers.
- Attach evidence.
- Close inspection.

---

## Process Life Cycle

The following steps represent the full functional lifecycle of an inspection:

1. The administrator defines an **inspection type**.
2. The administrator creates an **inspection** based on that type.
3. The inspection is **assigned** to a technician.
4. The technician **executes** the inspection in the field.
5. The technician **records answers** and evidence.
6. The system **validates** that the form is complete.
7. The inspection is **closed**.
8. The result is **stored** for historical consultation.

---

## Coherence with the rest of the system

This interpretation establishes the foundation for all other components of the project:

| Element | Justification |
|---------|--------------|
| Actors defined | Administrator and Technician drive all use cases. |
| Rules defined | Business rules directly shape the SQL constraints and validations. |
| Flows defined | The lifecycle maps directly to the flow diagrams. |
| SQL model | Structured entities (companies, users, inspections) live in SQL for process control. |
| NoSQL model | Dynamic forms and answers live in NoSQL due to their flexible, variable structure. |