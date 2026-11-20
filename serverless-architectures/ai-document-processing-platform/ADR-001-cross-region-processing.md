# ADR-001: Split storage and processing across two regions

**Status:** Accepted for the proof of concept

## Context
The platform handles supplier and compliance documents containing personal and business information, so the client wanted data at rest in South Africa to support POPIA requirements. At the time, the Bedrock text-generation models needed for extraction were not available in `af-south-1`.

## Options
1. **Everything in `eu-west-1`.** Simple and all services are available, but it gives up South African data residency.
2. **Everything in `af-south-1`.** Best for residency, but the required model was unavailable, so extraction could not be built.
3. **Storage in `af-south-1`, processing in `eu-west-1`.** Keeps the permanent record in South Africa and uses Ireland only for processing.

## Decision
Option 3.

- Uploads and final outputs stay in `af-south-1`.
- A temporary bucket and the OCR and extraction steps run in `eu-west-1`.
- Only the final CSV and audit metadata are written back to `af-south-1`.

## Consequences
- **Residency:** long-term storage remains in South Africa, but documents are copied to Ireland temporarily. This is a cross-border transfer that needs a documented lawful basis under POPIA section 72. Temporary copies are deleted in code once processing finishes. The analysis and open items are in [popia-considerations.md](./popia-considerations.md), and the client's compliance function should review them.
- **Engineering:** S3 event notifications can't target a function in another region, so the first design failed. The flow was redesigned so each trigger stays within one region, with an explicit copy step between regions.
- **Cost:** a small data-transfer cost for moving documents and results between regions.
- **Future:** when suitable models become available in `af-south-1`, the processing stage can move there and the temporary bucket can be removed.
