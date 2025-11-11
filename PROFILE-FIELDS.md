# Profile Fields Specification

## 1. Purpose
- Single source of truth for profile fields, validation, reciprocity mapping, and phasing to keep AI implementation simple.
- Aligns with TECH-STACK (frontend-first, Zod Week 3+, Cloudflare R2 media, vertical slices).

## 2. Condition Schema (standardized)
Use a consistent condition format:
```json
{ "field": "<field_id>", "op": "eq|ne|in|notIn|isSelected", "value": "<val>|[...]" }
```
Examples:
- { "field": "marital_status", "op": "ne", "value": "Unmarried" }
- { "field": "native_country", "op": "eq", "value": "India" }

## 3. Reciprocity Bundles
- education: educational_qualifications, education_in_detail
- occupation: occupation_category, occupation_in_detail, employment_category
- income: annual_income
- family: father_*, mother_*, siblings_* (counts)
- photos: profile_photo, album_photos, family_group_photos

## 4. Core Fields (Week 1-2)

| Field ID | Label | Type | Required (W1-2) | Validation (W1-2) | Notes |
|---|---|---|---|---|---|
| full_name | Full Name | text | yes | max 100 | |
| gender | Gender | radio | yes | Male/Female | |
| date_of_birth | Date of Birth | date_picker | yes | | age auto-derived |
| primary_mobile_number | Primary Mobile Number | phone_input | yes | OTP verify | single source for phone |
| email_id | Email ID | email | yes | unique | |
| create_password | Create Password | password | yes | min 8, max 14, complexity | |
| marital_status | Marital Status | dropdown | yes | Unmarried/Divorcee/Awaiting Divorce/Widower/Annulled | controls child counters |

Optional in W1-2 (move to W3+ as needed): complexion, body_type, physical_status, children counts, family_status.

Age rules: Female ≥18, Male ≥21 (derived from DOB).

## 5. Primary & Religious (Christian slice) (Week 3+ for most)

| Field ID | Label | Type | Required | Validation/Options | Notes |
|---|---|---|---|---|---|
| height | Height | dropdown | yes (W3+) | 134–213 cm | display: cm (ft'in'') |
| complexion | Complexion | dropdown | no | enum | |
| body_type | Body Type | dropdown | no | enum | |
| physical_status | Physical Status | dropdown | no | enum | children on Differently Abled |
| category_differently_abled | Category | multi_select_dropdown | cond | enum + "Other" → specify | |
| describe_differently_abled | Describe | textarea | cond | max 500 | |
| religion | Religion | dropdown | fixed | Christian | prefer deriving from route scope |
| denomination | Denomination | dropdown | yes | list | |
| diocese | Diocese | dropdown | yes | list + "Other" → diocese_name | |
| parish_name_place | Parish Name and Place | textarea | no | | |

### 5.1 Community-Specific (Tulunadu focus; phased)
| Field ID | Label | Type | Phase | Notes |
|---|---|---|---|---|
| caste_sub_caste | Caste/Sub-caste | dropdown | P0 (Hindu slice) | Bunt, Billava, Devadiga, Mogaveera, etc. |
| bunt_sub_community | Bunt Sub-communities | dropdown | P0 (Hindu slice) | Shetty, Hegde, Poojary, etc. |
| native_place | Native Place | text | P1 | Birth place (differs from current location) |
| mother_tongue | Mother Tongue | dropdown | P0 | Kannada, Tulu, Konkani, etc. (matching only) |
| religion_scoped | Religion Selection | derived | P0 | From route/subdomain; not user-editable in slice |
| community_selection | Community Selection | dropdown | P0 | Cascading by religion (where applicable) |
| tulunadu_region | Tulunadu Region | dropdown | P0 | Coastal Karnataka |
| local_festivals | Local Festivals | multi_select | P1 | Community-specific preferences |

## 6. Education & Professional (Week 3+; contributes to reciprocity)

| Field ID | Label | Bundle | Type | Required | Validation/Options |
|---|---|---|---|---|---|
| educational_qualifications | Educational Qualifications | education | dropdown | yes | options_ref: educational_qualifications_list |
| education_in_detail | Education in Detail | education | textarea | yes | max 200 |
| education_level | Education Level | education | dropdown | yes (P0) | High School, Diploma, Bachelor's, Master's, PhD, PG Diploma |
| education_field | Education Field/Subject | education | dropdown | P1 | Engineering, Business, Medicine, etc. |
| graduation_year | Graduation Year | education | dropdown | P1 | e.g., 2020–2030 |
| occupation_category | Occupation Category | occupation | dropdown | yes | options_ref: occupation_category_list |
| occupation_sector | Occupation Sector | occupation | dropdown | P0 | IT, Healthcare, Education, Finance, Engineering, Govt, Business, Legal, Defense, Others |
| occupation_in_detail | Occupation in Detail | occupation | textarea | yes | max 200 |
| job_title | Job Title/Role | occupation | text | P1 | |
| work_experience | Work Experience | occupation | dropdown | P1 | 0–1, 2–5, 5–10, 10+ years |
| employment_category | Employment Category | occupation | dropdown | yes | enum |
| working_country | Working Country |  | dropdown | yes | options_ref: country_list |
| work_location | Work Location |  | text | P0 | Current city/state |
| annual_income | Annual Income | income | dropdown | yes | enum continuous range incl "Prefer Not to Say" |

## 7. Location & Contact (Week 3+)

| Field ID | Label | Type | Required | Condition | Notes |
|---|---|---|---|---|---|
| native_country | Native Country | dropdown | yes |  | options_ref: country_list |
| native_state | State | dropdown | yes | {field:native_country, op:isSelected} | |
| native_district | District | dropdown | yes | {field:native_country, op:eq, value:India} | |
| mobile_number | Mobile Number | phone_input | yes |  | reuse from Step 1 (read-only) |
| whatsapp_number | WhatsApp Number | phone_input | no |  | |
| custodian_name | Custodian Name | text | cond | {field:created_by, op:ne, value:"Candidate"} | |
| custodian_relation | Custodian Relation | text | cond | same as above | |

Defer addresses (present/permanent) to later phase to reduce PII in MVP.

## 8. Profile Creation Details (Week 1-2)

| Field ID | Label | Type | Required | Notes |
|---|---|---|---|---|
| created_by | Created By | dropdown | yes | Candidate/Family/Relative/Friend etc. |
| creator_name | Creator Name | text | cond | required unless Candidate |
| creator_contact_number | Contact Number | phone_input | cond | required unless Candidate |
| how_did_you_hear_about_us | How Did You Hear About Us? | dropdown | yes | options_ref: how_did_you_hear_list |

## 9. Additional Family (Week 3+; contributes to reciprocity)

| Field ID | Label | Bundle | Type | Required |
|---|---|---|---|---|
| father_name | Father's Name | family | text | yes |
| father_native_place | Father's Native Place | family | text | yes |
| father_occupation | Father's Occupation | family | text | no |
| mother_name | Mother's Name | family | text | yes |
| mother_native_place | Mother's Native Place | family | text | yes |
| mother_occupation | Mother's Occupation | family | text | no |
| brothers_married | No. of Brothers (Married) | family | numeric | no |
| brothers_unmarried | No. of Brothers (Unmarried) | family | numeric | no |
| sisters_married | No. of Sisters (Married) | family | numeric | no |
| sisters_unmarried | No. of Sisters (Unmarried) | family | numeric | no |
| family_type | Family Type |  | dropdown | P1 | Nuclear/Joint |
| family_status | Family Status |  | dropdown | P2 | Lower/Middle/Upper/ Affluent |
| family_values | Family Values |  | dropdown | P2 | Traditional/Moderate/Liberal |
| manglik_status | Manglik Status |  | dropdown | P1 (Hindu) | Yes/No/Don't know/NA |

## 10. Photos (Week 3+; Cloudflare R2)
- profile_photo: image_upload (required before browsing others), client-side compression, max 15MB.
- album_photos: image_upload_multi (max 9, max 15MB each).
- family_group_photos: image_upload_multi (max 3, max 15MB each).
- privacy: Visible To All | Visible To Interest Sent or Accepted | Hide Photos (with sub-options).
- Storage: Vercel API → R2 private bucket; DB saves `storage_key`; access via signed URLs.

## 11. Partner Preferences (Optional at creation; add nudges later)
- All preference fields optional; UI supports “Any=Select All” behavior (not stored as literal “Any”).

## 12. Additional Personal (P0/P1/P2)
| Field ID | Label | Type | Phase | Notes |
|---|---|---|---|---|
| about_me | About Me / Bio | textarea | P0 | Max 500 (later 1000) with profanity filter |
| languages_known | Languages Known | multi_select | P0 | English, Hindi, Kannada, Tulu, Konkani, etc. |
| blood_group | Blood Group | dropdown | P1 | A+, A-, B+, B-, O+, O-, AB+, AB- |
| diet | Diet | dropdown | P1 | Vegetarian/Non-veg/Eggetarian/Vegan |
| smoking | Smoking | dropdown | P1 | Never/Occasionally/Regularly |
| drinking | Drinking | dropdown | P1 | Never/Socially/Regularly |
| residency_status | Residency Status | dropdown | P1 | Citizen/PR/Work Visa/Student Visa |
| citizenship | Citizenship | text | P1 | Country; dual citizenship possible |
| willing_to_relocate | Willing to Relocate | dropdown | P1 | Yes/No/Maybe/Within State/Abroad |
| weight | Weight | dropdown | P2 | 40–140 kg |
| spectacles | Spectacles | dropdown | P2 | Yes/No/Contact Lenses |
| institution_name | Institution Name | text | P2 | |
| grade_percentage | Grade/Percentage | text | P2 | |
| company_size | Company Size | dropdown | P2 | Startup/SME/MNC/Government |

## 12. Validation Strategy
- Week 1-2: HTML5 + simple TypeScript checks.
- Week 3+: Zod + React Hook Form schemas (one per step), aligned to reciprocity bundles.

## 13. Access & Reciprocity Summary
- Viewing other users’ education/occupation/income/family depends on having provided the corresponding bundle (after grace period), subject to plan rules.
- Premium plans may bypass reciprocity per admin default.

## 14. Notes for AI Implementation
- Build vertical slices page-by-page; start with Step 1 + minimal profile view.
- Use TypeScript constants for lists in Week 1-2; replace with DB-backed lists in Week 3+.
- Keep addresses and extended PII for a later phase to accelerate MVP and reduce risk.

---

## 15. Excluded Fields (MVP)
- Full communication/present/permanent addresses and PIN/ZIP (defer to later phase).
- Candidate asset details.
- Reference person (name/relationship/mobile) collection.
- Outstation leave dates.
- Residential landline number.
- Secondary mobile number (keep primary with OTP as source of truth).
- Video uploads and any rich media beyond photos.
- Government ID/document uploads for verification (post-MVP).

