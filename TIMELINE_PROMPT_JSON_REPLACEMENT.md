## Prompt replacement (Timeline Feature Upgrade)

Replace your current timeline instructions with the block below:

```text
### Timeline Engine (Longitudinal Oncology Event Model)

You must reconstruct a strict chronological cancer timeline across all documents.

For each encounter date, return one timeline entry and extract all qualifying events into an `events` array.

#### Required event types
1) DiagnosisConfirmation
- Capture the first definitive malignancy confirmation (pathology preferred).
- Fields:
  - diagnosisConfirmed: true/false
  - diagnosisType: string
  - confirmationMethod: Pathology | Cytology | Clinical
  - isPrimaryDiagnosisDate: true only once in the full case

2) CancerStaging
- Extract only explicitly documented staging.
- Fields:
  - stageType: Clinical | Pathologic
  - tnm: { t: string, n: string, m: string }
  - stageGroup: string

3) Treatment
- Capture treatment modality and details.
- Allowed treatmentType:
  - Oral
  - IVInfusion
  - Brachytherapy
  - Teletherapy
  - Surgery
- Fields:
  - treatmentType
  - agentsOrDetails
  - intent: Curative | Palliative | Unknown
  - sequence: Neoadjuvant | Adjuvant | Concurrent | Unknown
  - isTreatmentStart: true only for first treatment in full case

#### Timeline rules
- One timeline object per date.
- Multiple events can exist on the same date.
- Never merge separate event types into one object.
- Ordering must follow: Symptom -> Imaging -> DiagnosisConfirmation -> CancerStaging -> Treatment.
- Conflict resolution priority: Pathology > Imaging > Clinical.
```

## JSON replacement (Timeline Schema Upgrade)

Replace only the `timelineSummary` portion of your schema with this:

```ts
timelineSummary: z.array(
  z.object({
    date: z.string().describe("Encounter date in YYYY-MM-DD format."),
    summary: z.string().describe("2-5 line concise update for that date."),

    events: z.array(
      z.object({
        type: z.enum([
          "Symptom",
          "Imaging",
          "DiagnosisConfirmation",
          "CancerStaging",
          "Treatment",
          "FollowUp"
        ]),

        confidence: z.enum(["High", "Medium", "Low"]),
        sourcePriority: z.enum(["Pathology", "Imaging", "Clinical"]),

        data: z.object({
          // Diagnosis confirmation
          diagnosisConfirmed: z.boolean().optional(),
          diagnosisType: z.string().optional(),
          confirmationMethod: z.enum(["Pathology", "Cytology", "Clinical"]).optional(),
          isPrimaryDiagnosisDate: z.boolean().optional(),

          // Staging
          stageType: z.enum(["Clinical", "Pathologic"]).optional(),
          tnm: z
            .object({
              t: z.string().optional(),
              n: z.string().optional(),
              m: z.string().optional(),
            })
            .optional(),
          stageGroup: z.string().optional(),

          // Treatment
          treatmentType: z
            .enum(["Oral", "IVInfusion", "Brachytherapy", "Teletherapy", "Surgery"])
            .optional(),
          agentsOrDetails: z.string().optional(),
          intent: z.enum(["Curative", "Palliative", "Unknown"]).optional(),
          sequence: z.enum(["Neoadjuvant", "Adjuvant", "Concurrent", "Unknown"]).optional(),
          isTreatmentStart: z.boolean().optional(),
        }),

        evidence: evidenceArraySchema,
      })
    ),

    evidence: evidenceArraySchema,
  })
),
```
