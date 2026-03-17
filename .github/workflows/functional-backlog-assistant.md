---
name: Functional Backlog Assistant
on:
  workflow_dispatch:
    inputs:
      feature_title:
        description: "Name of the feature needing analysis"
        required: true
      feature_summary:
        description: "1-3 sentence summary of the problem or feature"
        required: true
      business_objective:
        description: "Business goal or success metric (optional)"
        required: false
      stakeholders_personas:
        description: "Key personas or user roles involved (comma-separated)"
        required: false
      constraints_dependencies:
        description: "Constraints, integrations, assumptions, or dependencies"
        required: false
      acceptance_focus:
        description: "Critical acceptance scenarios, edge cases, or risks to cover"
        required: false
      non_functional_notes:
        description: "Performance, security, compliance, or other NFR hints"
        required: false
      proposal_action:
        description: "Leave as 'propose' to get a draft for manual review; set to 'approve' after review to publish the final backlog"
        required: false
        default: "propose"
permissions:
  contents: read
  issues: read
  pull-requests: read
tools:
  github:
    mode: remote
    toolsets: [default]
network: defaults
safe-outputs:
  create-issue:
    max: 1

---

# Functional Backlog Assistant

Assist functional analysts by turning a provided feature description into a concise backlog of epics with functional analyses ready for QA and engineering. Always include, at minimum, **Title, Objective, Context, Personas, Description, Acceptance Criteria, and Non-functional requirements** for every epic. Treat the effort as **greenfield**: do not reuse existing repository content as a base.

## Inputs available
- `feature_title`, `feature_summary`
- Optional: `business_objective`, `stakeholders_personas`, `constraints_dependencies`, `acceptance_focus`, `non_functional_notes`
- `proposal_action` (default `propose`; set to `approve` only after manual review of the draft)
- Treat as greenfield: do **not** rely on repository documents as the starting point.

## Deliverables
1. 3–7 epics that together cover the requested feature while remaining independently testable and deliverable.
2. For each epic, provide all required sections using clear Markdown.
3. A dependency/sequence map and short delivery notes highlighting risks, assumptions, and open questions.
4. Publish the full backlog using `safe-outputs.create-issue` titled `Functional backlog: ${inputs.feature_title}` (substitute the workflow input) and include the Markdown deliverable **only when `proposal_action` is `approve`**. Otherwise, produce a draft for human review and stop without calling any safe outputs.

## Execution steps
1. **Understand the request**: Use only the workflow_dispatch inputs (do not reuse repository docs). If personas, constraints, or NFRs are missing, infer sensible defaults and clearly label assumptions.
2. **Decompose into epics**: Aim for 3–7 epics that are cohesive, independently shippable, and valuable. Keep scope per epic roughly 3–10 days of work.
3. **Functional analysis per epic**: For each epic, output the following subsections (all required):
   - **Title**
   - **Objective**
   - **Context** (dependencies, constraints, integrations)
   - **Personas** (bulleted)
   - **Description** (key workflows/user journeys and data notes)
   - **Acceptance Criteria** (use clear Given/When/Then bullets; cover success, edge, and error paths)
   - **Non-functional requirements** (performance, security, availability, audit, compliance, localization, accessibility, observability as applicable)
4. **Dependencies and sequencing**: Provide a simple dependency map and recommended implementation order.
5. **Manual review gate**: If `proposal_action` is not `approve`, output a draft (no safe outputs) and state that human approval is required. If `proposal_action` is `approve`, proceed to publish.
6. **Quality checks**: Ensure epics are not overlapping, acceptance criteria are testable, and NFRs are explicit rather than implicit.
7. **Output**: Present the backlog in Markdown with distinct sections:
   - `## Feature Summary`
   - `## Proposed Epics` (one subsection per epic with all required fields)
   - `## Dependencies & Delivery Notes` (dependency map, risks, assumptions, open questions)
   Respect the manual gate above: when `proposal_action` is `approve`, call `safe-outputs.create-issue` with the assembled Markdown and the title `Functional backlog: ${inputs.feature_title}`. Otherwise, provide the draft and exit without invoking safe outputs.

## Style
- Use crisp, unambiguous language.
- Keep acceptance criteria actionable and measurable.
- Call out any assumptions or missing information explicitly.
