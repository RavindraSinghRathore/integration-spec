# Integration Specifications Repository

**DXP Service Delivery - Integration Specifications**

This repository contains integration specifications for system-to-system integrations in our DXP (Digital Experience Platform) service delivery projects. Integration specs are created for client projects and maintained alongside solution code to enable AI-assisted development, documentation, and maintenance.

## 📁 Repository Structure

```
integration-spec/
├── specs/               # Integration specifications (one per integration)
├── patterns/            # Reusable integration patterns
├── docs/reference/      # Client reference documentation
├── .github/
│   ├── copilot-instructions.md    # AI agent behavior guidelines
│   ├── templates/                 # Integration spec template
│   └── prompts/                   # AI prompts for spec generation/review
└── README.md
```

## 🚀 Quick Start

### Creating a New Integration Spec

Use the GitHub Copilot prompt to generate a spec:

1. Open VS Code and activate Copilot Chat
2. Type: `#prompt:generate-integration-spec.prompt.md`
3. Follow the guided questions to provide integration details
4. The spec will be automatically generated in `specs/`

### Using Existing Patterns

Check the [`patterns/`](patterns/) directory for reusable integration blueprints:
- CMS to DAM integration
- Headless CMS delivery
- E-commerce integration
- And more...

### Adding Client Documentation

Store client-specific reference materials in [`docs/reference/{client-name}/`](docs/reference/):
- API documentation
- Architecture diagrams
- Data dictionaries
- Requirements documents

## 📝 Integration Spec Template

All integration specs follow the standard template located in [`.github/templates/integration-spec-template.md`](.github/templates/integration-spec-template.md).

### Key Sections

1. **Overview** - Business purpose and systems involved
2. **Business Requirements** - Functional requirements and stakeholders
3. **Technical Requirements** - Endpoints, protocols, authentication
4. **Integration Approach** - Architecture diagrams and patterns
5. **Data Mapping** - Field-level transformations
6. **Sample Payloads** - Request/response examples
7. **Error Handling** - Retry strategies and alerting
8. **Non-Functional Requirements** - SLAs, security, DXP considerations
9. **Testing Strategy** - Test scenarios and environments
10. **Deployment & Operability** - Deployment steps and runbooks
11. **Client Handoff & Support** - Responsibilities and escalation
12. **Open Items** - Tracked action items
13. **Approval** - Sign-off section

## 🔧 DXP-Specific Considerations

Our integration specs include dedicated sections for DXP service delivery:

### Multi-Channel Delivery
- Web, mobile, app, IoT, headless/API support
- Channel-specific content transformation

### Personalization & Analytics
- Customer data synchronization
- Tracking event integration
- Audience segmentation

### Content Delivery
- CDN strategy and cache invalidation
- Preview vs. published content modes
- A/B testing integration points

### Client Support
- Clear responsibility matrix (DXP team vs. client vs. vendor)
- Escalation paths and SLAs
- Client access to monitoring and logs

## 🤖 AI-Assisted Workflows

### Available Prompts

- **Generate Integration Spec**: `#prompt:generate-integration-spec.prompt.md`
- **Review Integration Spec**: `#prompt:review-integration-spec.prompt.md`
- **Update Integration Spec**: `#prompt:update-integration-spec.prompt.md`

### Copilot Instructions

The `.github/copilot-instructions.md` file provides AI agents with:
- DXP service company context
- Integration spec conventions
- DXP-specific considerations
- Multi-client project guidance

## 📋 File Naming Conventions

### Integration Specs
```
specs/{source-system}-to-{target-system}.md
```
Examples:
- `optimizely-cms-to-salesforce-commerce.md`
- `contentful-to-cloudinary-dam.md`
- `sitecore-to-coveo-search.md`
- `adobe-aem-to-marketo.md`

### Patterns
```
patterns/{pattern-name}.md
```
Examples:
- `cms-to-dam.md`
- `headless-cms-delivery.md`

## 🔐 Security & Confidentiality

⚠️ **Important:**
- Do not commit API keys, credentials, or tokens
- Sanitize client-specific sensitive information
- Use `.gitignore` patterns in `docs/reference/` for sensitive files
- Follow client confidentiality agreements

## 🤝 Contributing

### Adding New Patterns

When you create a reusable integration:

1. Generalize the solution by removing client-specific details
2. Create a pattern file in `patterns/`
3. Include architecture diagrams and data mappings
4. Update `patterns/README.md`

### Updating Templates

Template improvements should:

1. Maintain backward compatibility with existing specs
2. Be discussed with the team before implementation
3. Be documented in template comments

## 📚 Related Resources

- [Integration Patterns Library](patterns/)
- [Client Reference Documentation](docs/reference/)
- [Template Definition](.github/templates/integration-spec-template.md)
- [Copilot Instructions](.github/copilot-instructions.md)

## 🆘 Support

For questions or issues with integration specs:

- **Internal Team**: Contact the Integration Architecture team
- **AI Agent Issues**: Review `.github/copilot-instructions.md` for configuration
- **Template Updates**: Submit a pull request with proposed changes

---

**Last Updated:** April 22, 2026  
**Maintained By:** DXP Integration Team
