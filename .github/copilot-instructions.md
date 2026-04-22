# Copilot Instructions

## Project Context

This workspace contains integration specifications for system-to-system integrations in a **DXP (Digital Experience Platform) service delivery context**. Integration specs are created for client projects and live within the solution repository so that AI tooling can ingest, maintain, and evolve them alongside the codebase.

**Company Context:**
- DXP service-based company delivering digital experience solutions
- Multi-client environment with reusable integration patterns
- Platform-agnostic approach supporting various DXP, CMS, commerce, and marketing technology platforms
- Support both client-managed and vendor-managed system integrations

## Conventions

- Integration specs are stored in the `specs/` directory, one file per integration.
- File naming: `{source-system}-to-{target-system}.md` (lowercase, kebab-case).
- Reusable integration patterns are stored in `patterns/` directory.
- Client reference documentation goes in `docs/reference/` directory.
- Diagrams use Mermaid syntax and are embedded inline in spec files.
- All specs follow the template defined in `.github/templates/integration-spec-template.md`.
- Field-level data mappings use Markdown tables.
- Sample payloads use fenced code blocks with the appropriate language tag (json, xml, etc.).

## DXP-Specific Considerations

When generating specs for DXP integrations, always consider:

1. **Multi-Channel Delivery**: Web, mobile, app, IoT, headless/API
2. **Content Flow**: Authoring → Preview → Publish → Delivery
3. **Personalization**: Customer data synchronization, audience segmentation
4. **Analytics Integration**: Tracking events, conversion data, user behavior
5. **Asset Management**: DAM integration, CDN delivery, image optimization
6. **Caching Strategy**: CDN, edge caching, cache invalidation patterns
7. **Environment Complexity**: DEV, QA, UAT, Staging, Production, DR
8. **Client vs. Vendor Systems**: Ownership and support boundaries

## Agent Behavior

When generating or updating integration specs:

1. **Ask for client/project context** first: client name, engagement ID, project phase.
2. Always ask for business requirements, technical requirements, and the SA's integration approach before generating.
3. Apply the standard template — do not skip sections. Mark unknown sections as `TBD`.
4. Generate at least one sequence diagram and one data flow diagram per spec.
5. **Include DXP-specific section (8.4)** with multi-channel, personalization, CDN, and caching considerations.
6. **Define support responsibilities** clearly between DXP team, client, and vendors.
7. Place output files in `specs/`.
8. Check if a reusable pattern exists in `patterns/` before creating from scratch.
9. Validate that source/target systems, endpoints, and data mappings are consistent across the spec.
10. **Reference client documentation** in docs/reference/ if applicable.
