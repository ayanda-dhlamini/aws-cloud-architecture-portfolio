# POPIA Considerations

> This describes how the pipeline was designed to support POPIA requirements. It is a technical design record, **not legal advice and not a compliance certification**. Items marked **[VERIFY]** should be reviewed by the client's legal or compliance function before production use.

## Scope
The pipeline processes supplier and compliance documents (for example B-BBEE certificates, tax clearance certificates, invoices and quotes). These can contain personal information of individuals and information about companies. POPIA protects both.

## Design controls

| Concern | Control in this design |
|---|---|
| Data residency | Uploads and final outputs are stored in `af-south-1` (Cape Town). |
| Processing location | OCR and AI extraction run in `eu-west-1` (Ireland) because the required model was not available in Cape Town. Processing is transient. |
| Minimisation and retention | Temporary copies and intermediate OCR text in `eu-west-1` are deleted in code by a Lambda function once processing finishes. Only the extracted CSV and an audit file return to `af-south-1`. |
| Separation | Ingestion, temporary and output data are kept in separate buckets, with one output per document so customer data is not mixed. |
| Least privilege | Each Lambda has its own IAM role scoped to the buckets and actions it needs. |
| Audit | `metadata.json` records the document ID, file name, model, raw model output and timestamp. CloudWatch Logs cover the functions. |
| Failures | Failed documents are written to an `errors/` location for review. |

## Cross-border transfer (section 72)
Copying a document to Ireland is a transfer of information outside South Africa, so it needs a lawful basis. The grounds in section 72 include the recipient being subject to a law, binding corporate rules or a binding agreement that provides an adequate level of protection, or the data subject's consent, or necessity for performing a contract.

Points for the compliance review:
- Reliance on EU law alone may not settle it, because POPIA also protects companies while the GDPR does not.
- The usual practical route is a binding data-processing agreement with the provider. **[VERIFY]** which agreement covers this account and that the client is the responsible party and AWS an operator.
- **[VERIFY]** whether the data includes special personal information or children's information, which has additional rules.

## Items to close before production

1. **Textract service-improvement use.** By default, AWS may store and use content processed by Textract to improve the service, unless an AI services opt-out policy is applied through AWS Organizations. Apply the opt-out policy, and confirm the current AWS service terms.
2. **Model invocation logging.** Bedrock does not store or log prompts and completions by default. Keep model invocation logging disabled, or if it is enabled, store logs in `af-south-1` with encryption and retention limits.
3. **Inference profile routing.** The model is called through an inference profile that can route requests to other EU regions. Document the routing regions and include them in the transfer assessment.
4. **Deletion guarantee.** Deleting in code is correct, but failure paths can leave data behind. Add a short S3 lifecycle rule on the temporary bucket as a backstop and alarm on objects older than the expected processing time.
5. **Logs.** Confirm no document text or personal information is written to CloudWatch Logs in either region.
6. **Encryption.** Confirm default encryption on all buckets (SSE-S3 or KMS), TLS in transit and access logging. **[VERIFY]** the key choice.
7. **Retention.** Define how long the final CSVs and audit files are kept, and how deletion requests are handled.
8. **Incident response.** Document who is notified, and how, if a breach is suspected.
9. **Future state.** Revisit processing in `af-south-1` when a suitable model and OCR service are available there, which would remove the cross-border step.

## What this document does not claim
It does not claim the solution is POPIA compliant. Compliance depends on the client's policies, contracts and legal basis, which this repository does not contain.
