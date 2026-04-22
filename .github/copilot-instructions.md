# Copilot Instructions

## Project Context

This workspace contains integration specifications for system-to-system integrations. Integration specs live within the solution repository so that AI tooling can ingest, maintain, and evolve them alongside the codebase.

## Conventions

- Integration specs are stored in the `specs/` directory, one file per integration.
- File naming: `{source-system}-to-{target-system}.md` (lowercase, kebab-case).
- Diagrams use Mermaid syntax and are embedded inline in spec files.
- All specs follow the template defined in `.github/templates/integration-spec-template.md`.
- Field-level data mappings use Markdown tables.
- Sample payloads use fenced code blocks with the appropriate language tag (json, xml, etc.).

## Agent Behavior

When generating or updating integration specs:

1. Always ask for business requirements, technical requirements, and the SA's integration approach before generating.
2. Apply the standard template — do not skip sections. Mark unknown sections as `TBD`.
3. Generate at least one sequence diagram and one data flow diagram per spec.
4. Place output files in `specs/`.
5. Validate that source/target systems, endpoints, and data mappings are consistent across the spec.
