# ADR-002: Choosing the extraction model

**Status:** Accepted for the proof of concept

## Context
Extraction needed a model that could read OCR text and return type-specific structured fields. It had to run in `eu-west-1`, be cost-effective, and be an AWS-native model so billing stayed within AWS.

## Options considered
| Model | Outcome |
|---|---|
| Amazon Titan Text | Rejected. The first version reached end of life, and text generation was not available in the region. |
| Amazon Nova Micro | Rejected. Not available in the region at the time. |
| Amazon Nova 2 Lite | **Selected.** Available through a regional inference profile, AWS-native, low cost, and capable of structured extraction. |

## Decision
Use Amazon Nova 2 Lite through the Bedrock Converse API, with:
- Temperature 0 for deterministic output.
- A 2,048 token output limit, raised from 512 so detailed schemas fit.
- A two-step prompt: identify the document type, then extract the fields defined for that type.

## Consequences
- The model needs an **inference profile**, not on-demand throughput, and the profile can route requests to other EU regions. The IAM policy therefore needs region wildcards for the profile, which should be narrowed once routing is understood.
- Bedrock cost is a fraction of a cent per document at the token volumes seen, so OCR is the main cost.
- Extraction quality has only been checked on a sample. A labelled test set is needed before relying on it for production decisions.
- If a suitable model becomes available in `af-south-1`, the model choice and the region split should be revisited together (see ADR-001).
