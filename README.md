## Submission Video

[Watch video](https://www.loom.com/share/45c24b076d604ae8b1520d6b0cd80cc6)

---

# Diona Technologies — 2nd Assignment — Criminal Risk Assessment Request (XLSForm)

An ODK XLSForm recreating the Manitoba Families **"Criminal Risk Assessment Request"** PDF (2 pages) as a structured, mobile-ready digital data collection form.

## Assignment Overview

The objective of this assignment was to convert the provided paper/PDF form into a functional **ODK-compatible XLSForm**, while preserving the original form's fields, sections, required-field conventions, validation requirements, conditional logic, and relationships between fields.

The completed form is designed to make the paper-based process easier to complete digitally while maintaining the information and logic specified in the original document.

## Files

* `Criminal_Risk_Assessment_Request.xlsx` — The completed XLSForm containing:

  * `survey` sheet
  * `choices` sheet
  * `settings` sheet
---

# How a PDF Field Maps to an XLSForm Row

Every question on the paper form becomes **one row** in the `survey` sheet.

For a basic question, three columns are particularly important:

* `type` — defines what kind of answer is expected (`text`, `date`, `select_one`, etc.)
* `name` — the internal variable name used by the form and its logic
* `label` — the question or text displayed to the user

Multiple-choice questions do not contain their individual options directly in the `survey` sheet. Instead, the options are defined in the separate `choices` sheet.

For example:

```text
survey:
select_one gender

choices:
gender | male
gender | female
```

The `list_name` connects the question in the `survey` sheet with its available choices.

---

# Concepts Used in This Form

## 1. `begin group` / `end group`

The PDF contains clearly separated sections, including:

* Consent
* Subject Information
* Identification
* Assessment Request

These sections were represented using `begin group` and `end group` rows in the XLSForm.

The groups help keep related questions together and make the digital form easier to navigate.

The `field-list` appearance was used where appropriate so related questions can be presented together rather than forcing the user to move through every individual question separately.

---

## 2. `select_one` vs `select_multiple`

The appropriate XLSForm question type was selected based on how the options are represented in the original PDF.

### `select_one`

`select_one` allows the user to select only one option.

It is used for fields such as **Gender**, where the form presents mutually exclusive choices.

### `select_multiple`

`select_multiple` allows multiple options to be selected simultaneously.

It is used for:

* ID Types
* Reason for Risk Assessment

This matches the PDF's use of independent checkboxes, where more than one option can be applicable.

---

## 3. `relevant` — Conditional / Skip Logic

The `relevant` column was used for fields that should only appear when they are applicable.

For example, the **Other (specify ID)** field is displayed only when `Other` is selected in the ID-types question.

This is implemented using logic such as:

```text
selected(${idTypes}, 'other')
```

The same approach is used for fields such as the **MB Driver's Licence number**, where the field should only be displayed when the corresponding identification type has been selected.

The **Witness Signature** is also conditionally displayed based on the consent response.

This prevents users from seeing fields that are irrelevant to their selections.

---

## 4. `required` and `required_message`

The original PDF states:

> "NOTE – SECTIONS MARKED WITH AN ASTERISK (*) ARE REQUIRED"

The XLSForm follows this convention rather than arbitrarily making every field mandatory.

The fields explicitly marked as required in the PDF were configured with:

```text
required = yes
```

These include fields such as:

* Agency Name
* Reason for Assessment
* Assigned Worker
* Submitting Designate
* Designate Phone
* Designate Email
* Request Date

Required-field messages are also provided where appropriate to clearly communicate when a required response is missing.

---

## 5. `constraint` — Input Validation

Digital forms can perform validation that is not possible with a paper form.

The **Designate Email** field uses a regular-expression-based constraint to check that the entered value follows a valid email format.

A corresponding `constraint_message` informs the user when the entered value does not satisfy the expected format.

This provides basic data-quality validation before the form can be submitted.

---

## 6. `calculate` + `read_only`

The sample XLSForm provided as part of the assignment demonstrated the use of calculated values together with read-only fields.

That concept was adapted for the **Name of Person Being Assessed** field on page 2.

The PDF contains the instruction:

> "Must match information on page 1."

Instead of asking the user to type the person's name again, the form automatically constructs the name from the information entered on page 1.

The calculation follows the concept:

```text
concat(${firstName}, ' ', ${lastName})
```

The calculated value is then displayed using a read-only field.

This ensures that the name displayed on page 2 remains consistent with the information entered on page 1 and eliminates the possibility of a second manual entry containing a typo or mismatch.

---

## 7. `note` Type

Some content in the PDF is informational rather than something the user needs to answer.

Examples include:

* The legal consent paragraph
* CPIC/NICHE research information
* The disclaimer regarding criminal records checks

These sections were represented using the XLSForm `note` type.

A `note` displays information to the user without creating an input field.

This allows the important legal and instructional content from the original form to remain visible in the digital version.

---

## 8. `image` Type with `signature` Appearance

The original PDF contains physical signature fields.

These were implemented using:

```text
type: image
appearance: signature
```

This allows the user to capture a handwritten signature digitally rather than entering it as text.

The form includes signature capture for the relevant individuals, including:

* Person Being Assessed
* Witness

---

## 9. Consolidating Day / Month / Year into a `date` Field

The PDF displays Date of Birth using separate:

* Day
* Month
* Year

boxes.

For the digital form, these were represented using a single native XLSForm:

```text
type: date
```

This provides an appropriate date-selection interface while still capturing the complete date.

This is a deliberate adaptation to the digital medium rather than a literal recreation of the paper layout.

---

# Design Decisions

The form was designed with the following principles:

* **Preserve the original PDF's information**
* **Maintain the required/optional distinction**
* **Use appropriate XLSForm question types**
* **Avoid unnecessary duplicate data entry**
* **Use conditional logic where fields are context-dependent**
* **Validate important user input**
* **Keep informational/legal content visible**
* **Provide digital signature capture**
* **Make the form suitable for mobile data collection**

The goal was not simply to reproduce the appearance of the PDF, but to translate its structure and rules into a functional digital form.

---

# Assumptions

The following assumptions were made during implementation:

* Only fields explicitly marked with `*` in the original PDF are treated as required.
* "Unconsented" is represented using a `select_one yes_no` question rather than a standalone checkbox, as this is a standard XLSForm approach for a yes/no response.
* "Reason for Risk Assessment" is represented using `select_multiple` because the PDF presents the options as independent checkboxes rather than mutually exclusive choices.
* The Date of Birth fields were consolidated into a single `date` question to provide a more appropriate digital input mechanism.
* The page-2 "Name of Person Being Assessed" is automatically derived from the page-1 name fields to satisfy the PDF's requirement that the information must match.

---

## Final Deliverables

The repository contains the completed XLSForm and this README, along with the link to the video demonstration.

The XLSForm can be reviewed through its `survey`, `choices`, and `settings` sheets to understand the implemented questions, choices, validation, conditional logic, calculations, and form configuration.

