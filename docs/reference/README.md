# Client Reference Documentation

This directory stores client-specific reference materials that support integration specifications. These documents help AI agents understand client-specific context, requirements, and constraints when generating or maintaining integration specs.

## Directory Structure

```
docs/reference/
├── {client-name}/
│   ├── api-documentation/
│   ├── architecture-diagrams/
│   ├── data-dictionaries/
│   ├── requirements/
│   └── vendor-docs/
└── README.md
```

## What to Store Here

### API Documentation
- Vendor API specifications (OpenAPI/Swagger files)
- Custom API documentation from client systems
- Authentication/authorization guides

### Architecture Diagrams
- Client enterprise architecture diagrams
- Network topology diagrams
- System interaction diagrams
- Data flow diagrams

### Data Dictionaries
- Data model definitions
- Field mapping references
- Taxonomy and metadata schemas
- Code tables and lookup values

### Requirements Documents
- Business requirements documents (BRDs)
- Technical requirements documents (TRDs)
- Functional specifications
- Integration scope documents

### Vendor Documentation
- Third-party system documentation
- SaaS platform guides
- Integration quickstart guides
- Best practices documents

## File Naming Conventions

```
{client-name}/{category}/{system-name}_{document-type}_{version}.{ext}

Examples:
- acme-corp/api-documentation/salesforce_rest-api-reference_v58.pdf
- globex/data-dictionaries/product-catalog-schema_v2.xlsx
- initech/requirements/erp-integration-brd_draft-v3.docx
```

## Usage

When generating an integration spec:

1. **Check for relevant client documentation** in this directory
2. **Extract key information** (endpoints, data models, requirements)
3. **Reference source documents** in the generated spec
4. **Update this directory** when new documentation is received

## Maintaining This Directory

- Store only reference documents that support integration specs
- Remove outdated versions when superseded
- Keep a version history in file names
- Include a brief README in each client subfolder describing available materials

## Example Client Folder

```
docs/reference/acme-corp/
├── README.md (describes Acme Corp's systems and documentation)
├── api-documentation/
│   ├── commerce-cloud-api-v2.pdf
│   └── legacy-erp-soap-wsdl.xml
├── data-dictionaries/
│   ├── customer-data-model.xlsx
│   └── product-taxonomy.csv
└── requirements/
    └── cms-commerce-integration-brd.docx
```

## Security Note

⚠️ **Do not commit sensitive information:**
- API keys or credentials
- Production URLs with authentication tokens
- Customer PII or confidential data
- Proprietary client information without approval

Use `.gitignore` to exclude sensitive file patterns if needed.
