# Engagement Schema: Enterprise Governance / Admin

Copy this file to `<project>/personas/_schema.md` for engagements involving
enterprise governance, administration, or compliance-facing software. Creation
flows will prompt for every attribute declared here.

## Required attributes for this engagement's personas

```yaml
attributes:
  role_scope:           # what they administer: users, policies, billing, data
  permissions_level:    # global admin | delegated admin | auditor | end user
  compliance_context:   # SOC2, HIPAA, FedRAMP, internal audit...
  risk_posture:         # "blocks first, asks later" vs "enables, monitors"
  approval_chain:       # who they answer to; can they act unilaterally?
  org_size:             # seats / employees, rough is fine
  reference_tools:      # tools they use daily; expectations transfer from these
    - Okta
    - ServiceNow
```

## Why these matter in simulation

- `permissions_level` + `approval_chain`: a persona who cannot act
  unilaterally tests flows differently — they look for export, share, and
  review-for-approval paths, and their absence is a finding.
- `risk_posture`: shapes reaction to irreversible actions and missing
  confirmations.
- `reference_tools`: grounds transfer expectations, producing specific
  convention-violation findings ("expected this on the user detail page like
  Okta") instead of generic confusion.
- `compliance_context`: primes the persona to notice audit trails,
  attestation, and evidence-export gaps — the expert-lens findings section of
  the report.
