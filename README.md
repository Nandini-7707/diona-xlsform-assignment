# Diona Technologies — 2nd Assignment — Criminal Risk Assessment Request (XLSForm)

An ODK XLSForm rebuilding the Manitoba Families "Criminal Risk
Assessment Request" PDF (2 pages) as a structured, mobile-ready
data collection form.

## Files

- `Criminal_Risk_Assessment_Request.xlsx` — the XLSForm itself
  (3 sheets: `survey`, `choices`, `settings`)
- `build_xlsform.py` — the script used to generate it (kept for
  transparency/reproducibility, not required by the assignment)

## ⚠️ Before recording your video

1. Go to **https://getodk.org/xlsform/**, upload
   `Criminal_Risk_Assessment_Request.xlsx`, and confirm it
   converts with **no errors**. If it errors, read the error
   message (it'll point at a row/column) and fix that cell in the
   xlsx, then re-upload and re-check.
2. Read through the "Concepts used" section below a couple of
   times — this is what you'll need to explain **in your own
   words** on camera (their rule this round: no AI-scripted
   narration).

---

## How a PDF field maps to an XLSForm row — the core idea

Every question on the paper form becomes **one row** in the
`survey` sheet. Three columns matter most for a basic question:
- `type` — what kind of answer (text, date, select_one, etc.)
- `name` — the internal variable name (no spaces, used in logic)
- `label` — what the user actually sees on screen

Multiple-choice questions (radio buttons / checkboxes) don't list
their options in the `survey` sheet — options go in a **separate
`choices` sheet**, and the `survey` row just references a "list
name" that groups them. That's why you'll see e.g.
`select_one gender` in `type`, and then in `choices` there are two
rows both with `gender` in the `list name` column (`male`,
`female`).

## Concepts used in this form (explain these yourself, in your own words)

**1. `begin group` / `end group`**
The PDF has clear visual sections (Consent, Subject Information,
Identification, the page-2 Assessment Request). I mirrored each
section as a `group` in the form — this keeps related questions
together and lets ODK Collect show them as one screen
(`appearance: field-list` makes all questions in that group appear
on a single scrolling screen instead of one-question-per-screen).

**2. `select_one` vs `select_multiple`**
- `select_one` = only one answer possible (radio buttons) — used
  for Gender, since a person is either Male or Female on this form.
- `select_multiple` = more than one can be ticked at once
  (checkboxes) — used for the ID-types question (someone could
  provide a Birth Certificate *and* a Health Card), and for
  "Reason for Risk Assessment" (the PDF shows these as independent
  checkboxes, not a single choice).

**3. `relevant` (conditional/skip logic)**
Some fields on the PDF only make sense some of the time:
- "Other (specify ID)" text box only matters `relevant` (visible)
  if "Other" was actually selected in the ID-types question —
  written as `selected(${idTypes},'other')`.
- Same pattern for the MB Driver's Licence number field.
- The Witness signature is only needed if the person *did*
  consent, so it's `relevant` only when Unconsented = No.

This avoids showing irrelevant/confusing fields to the person
filling out the form on a tablet.

**4. `required` and `required_message`**
The PDF itself says: *"NOTE – SECTIONS MARKED WITH AN ASTERISK (*)
ARE REQUIRED"* on page 2. I only marked those exact fields as
`required = yes` in the form (Agency Name, Reason for Assessment,
Assigned Worker, Submitting Designate, Designate Phone, Designate
Email, Request Date) — matching the PDF's own required/optional
convention rather than guessing.

**5. `constraint` (validation)**
Designate Email has a `constraint` using a regex pattern that
checks it looks like a real email address, with a
`constraint_message` telling the user what's wrong if it fails.
This is basic input validation — the paper form can't stop someone
writing a broken email, but the digital form can.

**6. `calculate` + `read_only` (the technique borrowed from the sample form)**
The sample XLSForm they gave uses `calculate` rows to pull a value
(via `pulldata()`), then a visible `text`/`read_only` row to
*display* that calculated value to the user without letting them
edit it directly.

I reused that same pattern for a different purpose: page 2 of the
PDF has a field "Name of Person Being Assessed" with a note
*"Must match information on page 1."* Rather than making the user
re-type the name (risking a typo/mismatch), I used a `calculate`
row (`concat(${firstName}, ' ', ${lastName})`) to build the full
name automatically from the page-1 answers, then displayed it as
`read_only = true` on page 2. This guarantees the two pages always
match, which is exactly the requirement the PDF is asking for.

**7. `note` type**
The PDF has several large non-editable text blocks (the full legal
consent paragraph, the CPIC/NICHE research explanation, the final
disclaimer about not replacing a criminal records check). These
aren't questions, so I used the `note` type — it displays text to
the user with no input field, same as a static instructional block.

**8. `image` type with `signature` appearance**
Two fields on the PDF are physical signatures (person being
assessed, witness). ODK supports capturing a hand-drawn signature
via `type: image` with `appearance: signature`, which opens a
signature-capture pad instead of the camera.

**9. Consolidating Day/Month/Year into one `date` field**
The PDF prints Date of Birth as three separate boxes (Day / Month
/ Year). A native ODK `date` question already gives a proper date
picker covering all three in one control, so I used a single
`date` field rather than three separate `integer` fields — this is
a deliberate adaptation to the digital medium, not a 1:1 visual
copy of the paper layout.

## Assumptions

- Only the fields explicitly marked with `*` on page 2 are treated
  as required — page 1 has no asterisks in the source PDF, so
  those fields are left optional.
- "Unconsented" is modelled as a `select_one yes_no` rather than a
  bare checkbox, since XLSForm doesn't have a plain boolean
  checkbox type — `select_one yes_no` is the standard idiom for
  that.
- "Reason for Risk Assessment" is modelled as `select_multiple`
  (see Concepts #2) since the three options are printed as
  independent checkboxes on the PDF, not a single radio choice.
