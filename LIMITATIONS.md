# Platform Limitations and Considerations

This document outlines current limitations administrators and developers should consider when deploying and customizing the Redwood platform.

## Table of Contents
- [Overview](#overview)
- [Metadata and Data Validation Limitations](#metadata-and-data-validation-limitations)
  - [Study Metadata](#study-metadata)
  - [Data File Metadata](#data-file-metadata)
  - [Common Data Elements (CDEs)](#common-data-elements-cdes)
  - [Data Dictionary](#data-dictionary)
- [Content Management Limitations](#content-management-limitations)
  - [Database-Managed Content](#database-managed-content)
    - [Funding Opportunities](#funding-opportunities)
    - [News](#news)
    - [Events](#events)
    - [Newsletters](#newsletters)
    - [Other Prepopulated Data](#other-prepopulated-data)
  - [Frontend-Managed Content](#frontend-managed-content)
    - [Resource Center](#resource-center)
- [Related Documentation](#related-documentation)


## Overview

This document summarizes platform limitations that affect configuration, data and metadata validation, and content updates in Redwood. It clarifies which changes can be made through existing interfaces or database updates, and which require code changes or updates to seed scripts. Use this guide to scope effort, plan data onboarding, and understand current constraints before deployment or customization.


## Metadata and Data Validation Limitations

### Study Metadata

The study metadata template is currently **predefined and fixed** within the Redwood platform. During study registration, users are required to complete this standardized form to provide the required study metadata.

Some metadata fields rely on **controlled terms** (for example, enumerated values). While this approach promotes consistency and interoperability across studies, it may **not fully support all project-specific or domain-specific use cases**. In some cases, the available controlled terms may be too restrictive or not sufficiently expressive.

At this time, users **cannot customize the study registration form** or modify the controlled-term lists. This limits flexibility for teams that require additional metadata fields, alternative terminologies, or different metadata structures.

**Future Work**  
An enhancement is to allow users to **define and manage their own study registration forms**, including:
- Adding or removing metadata fields  
- Customizing controlled vocabularies   

Until these features are available, users are encouraged to review the predefined [study metadata template](https://openview.metadatacenter.org/templates/https:%2F%2Frepo.metadatacenter.org%2Ftemplates%2F9ade43b0-ea0c-499d-ac65-4b624a3b9db4) to ensure it aligns with their essential requirements before using the platform.

---
### Data File Metadata
Similar to study metadata, [the data file metadata template](https://openview.metadatacenter.org/templates/https:%2F%2Frepo.metadatacenter.org%2Ftemplates%2Fc691629c-1183-4425-9a12-26201eab1a10) is **predefined and fixed** within the Redwood platform. The template defines the required structure, fields, data types, and controlled terms that must be used when describing data files.

Data file metadata is applied **at the time of data upload**. When a data file is uploaded together with its corresponding metadata file, the platform performs a **strict validation** of the metadata file against the predefined template. This validation ensures that:
- The metadata file is a valid JSON document  
- All required metadata fields are provided  
- Field values conform to the expected data formats  
- Controlled-term fields use values from the predefined vocabulary lists  

If the metadata file does not comply with the template, it **will fail validation** and the data file **cannot be uploaded successfully** to the platform.

Because the predefined data file metadata template is built into the validator and cannot currently be customized, this approach may limit flexibility for projects with specialized metadata requirements or alternative data models.

**Future Work**  
Enhancements include enabling users to **customize the data file metadata template**, allowing them to:
- Define project-specific metadata fields  
- Extend or replace controlled vocabularies   

Until customization is supported, users should ensure that their data file metadata strictly follow the predefined template to avoid upload failures.

### Common Data Elements (CDEs)
**Common Data Elements (CDEs)**: A type of health data specification commonly used in clinical and research settings to capture and bind together complex phenomena, like depression, through standardized, consistent, well-defined questions (variables), paired with a description of allowable responses (values or value type) that are used in a standardized, machine-readable manner across studies or trials to prevent avoidable variability.

The platform uses a [**Global Codebook**](./Global_Codebook.xlsx), which serves as the authoritative data dictionary for all required CDEs. The Global Codebook defines the structure, semantics, and permissible values for each CDE.

When a data file is uploaded with a filename ending in `"transformcopy"`, it is treated as a **harmonized data file**. For such files, the platform backend automatically performs **CDE validation**, which enforces the following rules:
- The harmonized data file must contain **at least one CDE** defined in the Global Codebook  
- All CDE values must conform to the definitions and allowable values specified in the Global Codebook  

Because CDE validation is strictly based on the Global Codebook, all harmonized data files must use CDEs defined in that codebook. Projects that maintain their own Common Data Elements—such as domain-specific variables, alternative definitions, or locally governed value sets—cannot currently validate or upload data using those custom CDEs.

At present, the platform does not support customized harmonization workflows or project-specific CDE definitions. All harmonized data must be mapped to the Global Codebook in order to pass CDE validation. This may require projects to adapt or restructure their data, even when their existing CDEs are well defined and internally consistent.

**Future Work**  
An enhancement is to support **customizable harmonization**, allowing projects to define and manage their own CDEs. This would include enabling project-specific codebooks, validation rules, and harmonization logic, while preserving consistency and interoperability within each project’s context.

### Data Dictionary

When a data dictionary is uploaded together with its corresponding data file, the platform performs a **backend validation** of the data dictionary. To pass this validation, the data dictionary must conform to a **specific CSV-based format** defined by the platform.

The required data dictionary specification, including expected structure, column definitions, and formatting rules, is documented [here](https://github.com/bmir-radx/radx-data-dictionary-specification/blob/main/radx-data-dictionary-specification.md). Data dictionaries that do not follow this specification will fail validation and cannot be uploaded successfully.

Because the data dictionary format is strictly enforced and not currently configurable, this approach may limit flexibility for projects that use alternative data dictionary structures or formats.

**Future Work**  
An enhancement is to support more flexible data dictionary ingestion, including the ability to accommodate project-specific formats or configurable validation rules, while maintaining consistency and data quality checks.

## Content Management Limitations

### Database-Managed Content

Several content types are stored in the database and can be managed without code changes. However, they still require direct database access to modify.

For detailed information about the relevant database tables and their structure, see the [database schema creation scripts](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/02_create_base_db.sql).

### Funding Opportunities

**Location:** Database table `public.funding`

**Current Implementation:**

Funding opportunities are stored in the database and displayed on:
- Homepage (Funding Opportunities section)
- Funding Opportunities page (`/fundingOpportunities`)

**SQL Example - Insert New Funding Opportunity:**

```sql
-- Insert a new funding opportunity
INSERT INTO public.funding (
    slug,
    title,
    description,
    notice_number,
    activity_code,
    url,
    release_date,
    expiration_date,
    created_by
) VALUES (
    'nih-research-grant-2026',
    'NIH Research Grant Program for Data Science',
    'This funding opportunity supports innovative research in data science and computational biology. Applicants are encouraged to propose projects that leverage large-scale data analysis.',
    'NOT-OD-26-015',
    'R01',
    'https://grants.nih.gov/grants/guide/notice-files/NOT-OD-26-015.html',
    '2026-02-01',
    '2026-06-30',
    9999
);
```

**API Endpoints:**
- `GET /api/entity/v1/getFunding` - Returns current/active funding opportunities
- `GET /api/entity/v1/getAllFunding` - Returns all funding opportunities

### News

**Location:** Database tables `public.news` and `public.news_link`

**Current Implementation:**

News articles are stored in the database and displayed on:
- Homepage (Latest News & Updates section)
- News page (`/news`)
- Individual news article pages (`/news/[slug]`)

**Important Notes:**

- The `slug` must be unique and URL-safe (use lowercase, hyphens, no spaces)
- News items are filtered by `type_id` (1 = "general news")
- Active news is determined by current date being between `start_date` and `expiration_date`
- Description can include `|||` separator for truncation in listings (see NewsMapper in backend)
  - Format: `Short description ||| Extended content ||| Additional details`
  - The homepage shows only the first part before `|||`

**SQL Example - Insert New News Article:**

```sql
-- First, insert the news article
INSERT INTO public.news (
    slug,
    title,
    description,
    type_id,
    start_date,
    expiration_date,
    created_by,
    archived
) VALUES (
    'data-hub-upgrade-2026',
    'Redwood Announces Major Platform Upgrades',
    'We are excited to announce significant improvements to the Redwood platform, including enhanced search capabilities and new analytical tools. ||| These upgrades provide researchers with more dynamic access to explore datasets and utilize integrated analysis tools such as Jupyter notebooks, R, and Python. ||| The platform now supports real-time collaboration features and improved data visualization.',
    1,  -- 1 = "general news" type
    '2026-01-15',
    '2026-06-30',
    9999,
    false
);

-- Optional: Add related links to the news article
INSERT INTO public.news_link (
    news_id,
    url,
    text
) VALUES 
(
    (SELECT id FROM public.news WHERE slug = 'data-hub-upgrade-2026'),
    'https://example.org/upgrade-details',
    'Read Full Announcement'
),
(
    (SELECT id FROM public.news WHERE slug = 'data-hub-upgrade-2026'),
    'https://example.org/user-guide',
    'View Updated User Guide'
);
```
**API Endpoints:**
- `GET /api/entity/v1/getNews` - Returns recent news for homepage
- `GET /api/entity/v1/getAllNews` - Returns all active news
- `GET /api/entity/v1/getNews/[slug]` - Returns specific news article

### Events

**Location:** Database tables `public.events` and `public.event_link`

**Current Implementation:**

Events are stored in the database and displayed on:
- Homepage (Save the Date section)
- Events page (`/events`)

**Important Notes:**

- Events are automatically filtered by `event_date` (upcoming vs. past)
- Past events are displayed separately on the Events page
- Homepage shows only the next 3 upcoming events
- Use `event_type_id` to categorize events (e.g., webinar, workshop, conference)


**API Endpoints:**
- `GET /api/entity/v1/getEvents` - Returns upcoming events (next 3)
- `GET /api/entity/v1/getAllEvents` - Returns all events (current and past)

### Newsletters

**Location:** Database table `public.newsletter`

**Current Implementation:**

Newsletters are stored in the database and displayed on the Newsletters page (`/newsletters`). Newsletters are grouped by year based on their release date.

**SQL Example - Insert Newsletter:**

```sql
-- Insert a newsletter for Q1 2026
INSERT INTO public.newsletter (
    title,
    url,
    release_date,
    created_by
) VALUES (
    'Redwood Newsletter - Q1 2026',
    'https://s3.amazonaws.com/your-bucket/newsletters/2026-Q1-newsletter.pdf',
    '2026-03-31',
    9999
);
```

**API Endpoint:**
- `GET /api/entity/v1/getNewsletters` - Returns all published newsletters grouped by year

### Other Prepopulated Data

Lookup tables (`lkup_*`) are prepopulated by the database seed scripts in `/datahub-development/db/postgres/db-create-scripts/`:
- [`03_populate_base_tables.sql`](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/03_populate_base_tables.sql) - Base reference data used by the platform
- [`04_populate_variable_tables.sql`](https://github.com/bmir-datahub/datahub-development/blob/feature/aws/db/postgres/db-create-scripts/04_populate_variable_tables.sql) - Variable-related reference data derived from the Global Codebook

These values are not managed through the UI. To add or update lookup values, you must edit the seed scripts and reapply them in the target environment.

**Example - Add a new entity type**

If you need an additional entity type (for example, `video` beyond the default `study`, `document`, `image`, `variable`, `datafile`), add a row to `public.lkup_entity_type`:

```sql
INSERT INTO public.lkup_entity_type (id, name, description)
VALUES (6, 'video', NULL);
```
---

### Frontend-Managed Content

#### Resource Center

**Current Implementation:**

On the Redwood website, the **Helpful Information → Resource Center** section displays content as cards grouped into predefined categories: General, For Researchers, and For Submitters. The content of the cards is **hardcoded directly in the frontend codebase**. This means:

- Card content, titles, descriptions, and buttons are defined in JSX files
- Changes require editing JavaScript/React component files
- Updates require rebuilding and redeploying the frontend application
- No admin interface exists for managing these cards

**Editing cards and categories:**

At present, there is no UI-based admin interface for editing cards or managing categories. To add, modify, or remove cards or categories, changes must be made directly in the frontend code.

For detailed implementation steps and examples, refer to: [Resource Center Customization Guide](./RESOURCE_CENTER_CUSTOMIZATION.md)

---

## Related Documentation

- [Resource Center Customization Guide](./RESOURCE_CENTER_CUSTOMIZATION.md) - Detailed guide for editing Resource Center cards
- [Deployment Guide](./DEPLOYMENT_GUIDE.md) - Platform deployment instructions
---

**Last Updated:** January 2026  
**Version:** 1.0
