---
---

# TOGAF (The Open Group Architecture Framework)

TOGAF is a high-level framework for enterprise architecture which provides an approach for designing, planning, implementing, and governing an enterprise information technology architecture. It is the most used framework for Enterprise Architecture (EA) as of 2025.

## Core Concepts: ADM (Architecture Development Method)

The **Architecture Development Method (ADM)** is the core of TOGAF. It provides a tested and repeatable process for developing architectures. It is an iterative cycle that consists of several phases.

### **ADM Cycle Phases**

1. **Preliminary Phase**: Defines the "where, what, why, who, and how" of the architecture. It establishes the architecture capability and defines the principles.
2. **Phase A: Architecture Vision**: Sets the scope, constraints, and expectations for a project. It identifies stakeholders and creates the Architecture Vision document.
3. **Phase B: Business Architecture**: Describes the product/service strategy, and the organizational, functional, process, information, and geographic aspects of the business environment.
4. **Phase C: Information Systems Architectures**:
    * **Data Architecture**: Defines the major types and sources of data necessary to support the business.
    * **Application Architecture**: Defines the kinds of application systems necessary to process the data and support the business.
5. **Phase D: Technology Architecture**: Describes the software and hardware infrastructure needed to support the deployment of business, data, and application services.
6. **Phase E: Opportunities & Solutions**: Initial implementation planning and the identification of delivery vehicles for the architecture defined in previous phases.
7. **Phase F: Migration Planning**: Addresses how to move from the Baseline to the Target Architecture by finalizing a detailed Implementation and Migration Plan.
8. **Phase G: Implementation Governance**: Provides architectural oversight of the implementation.
9. **Phase H: Architecture Change Management**: Establishes procedures for managing change to the new architecture.
10. **Requirements Management**: A continuous process at the center of the ADM cycle that ensures all requirements are identified, stored, and addressed.

## Deliverables and Artifacts

TOGAF distinguishes between deliverables, artifacts, and building blocks.

### **Building Blocks**
- **Architecture Building Blocks (ABBs)**: Capture architectural requirements and direct and guide the development of SBBs. They are typically abstract (e.g., "Customer Database").
- **Solution Building Blocks (SBBs)**: Represent components that will be used to implement the required capability (e.g., "PostgreSQL Cluster v15").

### **Artifacts**
Artifacts are specialized documents that describe an aspect of the architecture. They are categorized into:
- **Catalogs**: Lists of things (e.g., Application Catalog).
- **Matrices**: Show relationships between things (e.g., Data/Application Matrix).
- **Diagrams**: Visual representations (e.g., Process Flow Diagram).

### **Architecture Repository**
The **Architecture Repository** is a logical information model used to store different classes of architectural output at different levels of abstraction. It includes the Architecture Metamodel, Architecture Landscape, and Standards Information Base.

## Relevance in Enterprise Architecture

TOGAF is considered the "Gold Standard" for Enterprise Architecture because:
1. **Vendor Neutrality**: It can be used by any organization regardless of their technology stack.
2. **Flexibility**: The ADM is modular and can be adapted to specific organizational needs.
3. **Standardization**: Provides a common language (Taxonomy) for architects.
4. **Ecosystem**: Extensive certification program and a large community of practitioners.

## Interview Preparation

### **Standard Interview Questions**

1. **What is the ADM and why is it important?**
   - **Answer**: The Architecture Development Method (ADM) is a repeatable process for developing architectures. It is important because it provides a structured, multi-phase approach that ensures all domains (Business, Data, Application, Technology) are addressed and aligned with business goals.

2. **Compare Phase B (Business Architecture) and Phase D (Technology Architecture).**
   - **Answer**: Phase B focuses on the business strategy, organization, and processes (the "What" and "Who"). Phase D focuses on the technical infrastructure, hardware, and software required to support those business processes (the "How" and "With What").

3. **What is the difference between an ABB and an SBB?**
   - **Answer**: An Architecture Building Block (ABB) is an abstract specification of a capability (e.g., "Identity Management System"). A Solution Building Block (SBB) is the specific implementation or product used to fulfill that capability (e.g., "Okta" or "Keycloak").

4. **How does Requirements Management fit into the ADM?**
   - **Answer**: Requirements Management is not a sequential phase but a continuous process located at the center of the ADM. Every other phase interacts with it to ensure that the architecture being developed remains aligned with stakeholder requirements.

5. **What is the Architecture Repository?**
   - **Answer**: It is a central storage for all architecture-related information, including previous versions of architectures, standards, governance documents, and building blocks. It facilitates reuse and consistency across projects.
