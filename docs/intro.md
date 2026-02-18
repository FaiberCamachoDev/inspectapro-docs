---
id: intro
title: Introduction
sidebar_position: 1
---

# Welcome to InspectaPro Docs

**InspectaPro** is a SaaS platform designed for companies that perform operational field inspections — covering areas such as technical maintenance, internal audits, quality control, inventory verification, and industrial safety.

This documentation covers the full architectural and business design of the system, including data modeling decisions, system actors, process flows, and the justification behind a hybrid SQL + NoSQL data architecture.

---

## Teammates

| # | Name |
|---|------|
| 1 | Faiber Camacho |
| 2 | Ximena Jaramillo |
| 3 | Cesar Rios |
| 4 | Samuel Cardona |

---

##  How to navigate this documentation

Use the **left sidebar** to move between sections. The documentation is organized in the following order:

| Section | Description |
|---------|-------------|
|  Business Interpretation | Business model, actors, rules, and lifecycle |
|  Use Cases | Complete use case diagram and descriptions |
|  Flow Diagrams | Inspection lifecycle with states and decision points |
|  Relational Model (SQL) | Normalized ERD and entity justifications |
|  Document Model (NoSQL) | JSON document structure for dynamic inspection data |
|  Architectural Justification | Why hybrid, risks, and how both models connect |

>  **Tip:** Each section is self-contained but all sections are coherent with each other — business rules connect to the SQL model, which connects to the NoSQL model, which connects to the architectural justification.

---

##  Purpose of this project

This project goes beyond simple CRUD thinking. It demonstrates:

- How to identify what data is structured vs. dynamic.
- How normalization applies in real business contexts.
- How document modeling supports flexibility and evolution.
- How business requirements shape data architecture decisions.