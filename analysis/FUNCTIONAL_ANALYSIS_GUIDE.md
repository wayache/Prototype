# Functional Analysis Guide for Agentic SDLC Workflow

## Overview
When a feature is submitted for functional analysis, your role is to break it down into **logical, independently developable epics** and create a detailed functional analysis document for each epic.

## Process

### Step 1: Feature Decomposition
- Identify the main feature/capability requested
- Break it into distinct **Epics** based on logical business domains or technical modules
- Each Epic should represent a cohesive piece of functionality that can be developed independently
- Ensure Epics can be tested and deployed separately

### Step 2: Create Functional Analysis per Epic
For each Epic, create a document with the following sections:

### Functional Analysis Document Template

```markdown
# [Epic Name]

## Title
[Descriptive title for this epic]

## Objective
[What this epic aims to achieve - the core business goal]

## Context
[Background information, dependencies on other systems, existing constraints]

## Personas
[List of users/roles who interact with this epic - e.g., Doctor, Nurse, Patient, Admin]

## Description
[Detailed explanation of what this epic does, including:
- Main workflows
- User journeys
- Key interactions
- Data flows]

## Acceptance Criteria
[Specific, testable conditions that must be met:
- Scenario 1: Given... When... Then...
- Scenario 2: Given... When... Then...
- Edge cases and error scenarios]

## Non-Functional Requirements
[Technical requirements not directly part of acceptance criteria:
- Performance (e.g., response time < 500ms)
- Scalability (e.g., handle 1000+ concurrent users)
- Availability (e.g., 99.9% uptime)
- Security (e.g., HIPAA compliance)
- Data retention
- Audit logging]
```

## Example Structure
For a **Patient Queue System** feature, typical epics might be:

1. **Epic: Patient Queue Management**
   - Objectives: Add/remove patients from queue, view queue status
   
2. **Epic: Doctor Availability Detection**
   - Objectives: Track doctor status, mark available/unavailable

3. **Epic: Automatic Patient-Doctor Assignment**
   - Objectives: Match available doctors to queued patients based on rules

4. **Epic: Queue Notifications**
   - Objectives: Alert patients and doctors of assignments/changes

5. **Epic: Queue Reporting & Analytics**
   - Objectives: Track wait times, assignment metrics, system performance

## Guidelines

- **Independence**: Each epic should be independently valuable and testable
- **Scope**: Keep epics focused - not too large (>2 weeks dev), not too small (<1 day dev)
- **Clarity**: Use clear language avoiding ambiguity
- **Completeness**: Ensure all acceptance criteria are specific and measurable
- **Non-functional Requirements**: Don't assume defaults; be explicit about performance, security, and scalability needs

## Deliverables

For each feature request, provide:
1. List of identified Epics
2. One functional analysis document per Epic
3. Dependency map showing how Epics relate to each other
