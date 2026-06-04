# Demo Data Setup: Centers & Affiliation Institutions

This guide explains how the seeded **Centers** and **affiliation institutions** are defined, and how to replace the neutral demo values shipped with Canopy with the real ones for your deployment.

The database init scripts ship with **generic placeholder** Centers (`Center A`–`Center E`) and institutions (`Demo University`, etc.) so that a fresh install contains no organization-specific data. Before going live, swap these for the names your installation actually uses.

**Depends on:** the database schema deployed from `canopy-cli/assets/db/postgres/init/` (Step 7a in the [Deployment Guide](./DEPLOYMENT_GUIDE.md)). All file paths below are relative to that `init/` directory.

## Table of Contents
- [Overview](#overview)
- [What the Demo Data Contains](#what-the-demo-data-contains)
- [Centers](#centers)
  - [Where Centers Live](#where-centers-live)
  - [The 710 ↔ 706 Sync Requirement](#the-710--706-sync-requirement)
  - [Renaming a Center](#renaming-a-center)
  - [Adding a Center](#adding-a-center)
  - [Removing a Center](#removing-a-center)
- [Affiliation Institutions](#affiliation-institutions)
  - [Where Institutions Live](#where-institutions-live)
  - [Adding / Editing an Institution](#adding--editing-an-institution)
  - [ROR IDs](#ror-ids)
- [Applying Your Changes](#applying-your-changes)
  - [Fresh Install](#fresh-install)
  - [Existing Environment (Incremental)](#existing-environment-incremental)
- [Out of Scope](#out-of-scope)

---

## Overview

There are two kinds of organization data seeded at install time, each in its own file under `canopy-cli/assets/db/postgres/init/`:

| Concept | What it is | Files |
|---|---|---|
| **Center** | The top-level program/grouping a study and a user belong to (`users.center_id`, study "Center" facet). | `710_data_lkup_center.sql`, `706_data_lkup_property_codelist_value.sql` (codelist `2`) |
| **Affiliation institution** | The organizations users can list as their affiliation (`users.institution_id`). | `900_seed_test_data.sql` |

Both are seeded with neutral placeholders. Edit the SQL files in place and they take effect on the next schema load (fresh install) or when applied manually (existing environment).

---

## What the Demo Data Contains

Out of the box:

**Centers** (`710` and the matching `706` codelist `2`):

| id | Demo value |
|---|---|
| 1 | Center A |
| 2 | Center B |
| 3 | Center C |
| 4 | Center D |
| 5 | Center E |

**Institutions** (`900`):

| id | Demo value | Type |
|---|---|---|
| 1 | Demo University | Academic Institution |
| 2 | Example Medical School | Academic Institution |
| 3 | Sample Government Agency | Government Agency (NIH) |
| 4 | Demo Health System | Hospital and Healthcare |

> ⚠️ **Note:** The demo institutions ship with `ror_id` set to `NULL` on purpose — a [ROR ID](#ror-ids) uniquely identifies a real-world organization, so leaving a real one in place would not be "neutral." Supply your own when you add real institutions.

---

## Centers

### Where Centers Live

A Center is stored in **two places that must agree**:

1. **`710_data_lkup_center.sql`** — the `lkup_center` table. This is the canonical list. `users.center_id` is a foreign key into it.
   ```sql
   INSERT INTO public.lkup_center VALUES (1, 'Center A', NULL);
   --                                      id   name      description
   ```
2. **`706_data_lkup_property_codelist_value.sql`** — codelist `2` (`'Center'`, key `study_center`). This is the **study-facing picklist** used to tag and filter studies by Center. These rows are matched to studies **by name (string)**, not by id.
   ```sql
   INSERT INTO public.lkup_property_codelist_value VALUES (8, 2, 'Center A', 40);
   --                                                       id  cl  value     display_order
   ```

### The 710 ↔ 706 Sync Requirement

Because studies are tagged with the Center **name** (the codelist value, not the `lkup_center` id), **the display names in `710` and in `706` codelist `2` must match exactly.** If they drift, a study's Center facet will not line up with the canonical Center list.

The shipped mapping (note the two files use independent ids — only the *names* must match):

| `710` id | `706` codelist-2 id | Shared name |
|---|---|---|
| 1 | 8  | Center A |
| 2 | 7  | Center B |
| 3 | 5  | Center C |
| 4 | 6  | Center D |
| 5 | 363 | Center E |

### Renaming a Center

Edit the name string in **both** files for that Center. Example — rename `Center A` to `Coordinating Center`:

```sql
-- 710_data_lkup_center.sql
INSERT INTO public.lkup_center VALUES (1, 'Coordinating Center', NULL);

-- 706_data_lkup_property_codelist_value.sql  (codelist 2 row with the matching name)
INSERT INTO public.lkup_property_codelist_value VALUES (8, 2, 'Coordinating Center', 40);
```

### Adding a Center

Add one row to each file. Pick a fresh, unused `id` in **each** file (they don't have to match across files), and the next free `display_order` in `706`:

```sql
-- 710_data_lkup_center.sql
INSERT INTO public.lkup_center VALUES (6, 'New Center', NULL);

-- 706_data_lkup_property_codelist_value.sql  (codelist 2)
INSERT INTO public.lkup_property_codelist_value VALUES (400, 2, 'New Center', 60);
```

> ⚠️ **Note:** `706` ids are shared across **all** codelists in that file, so choose an id not used by any other row (not just within codelist `2`). The `800_sequence_resets.sql` step realigns sequences to `MAX(id)` after load, so high ids are fine.

### Removing a Center

Delete the corresponding row from **both** files. Do not remove a Center that an existing user or study still references — reassign those records first, or the foreign key load / app queries will break.

---

## Affiliation Institutions

### Where Institutions Live

Seeded institutions are in **`900_seed_test_data.sql`**, inserted into the `institution` table. Each record is a block like:

```sql
(
    1,                  -- id
    'Demo University',  -- name
    23,                 -- status_id           (lkup_status)
    10,                 -- institution_type_id (lkup_institution_type)
    false,              -- is_for_profit
    NULL,               -- ror_id
    1,                  -- country_id          (lkup_country)
    5,                  -- state_id            (lkup_state)
    NULL,               -- province_region
    1                   -- created_by
),
```

The lookup id columns resolve against:

| Column | Lookup table | Seed file |
|---|---|---|
| `status_id` | `lkup_status` | `721_data_lkup_status.sql` |
| `institution_type_id` | `lkup_institution_type` | `713_data_lkup_institution_type.sql` |
| `country_id` | `lkup_country` | `708_data_lkup_country.sql` |
| `state_id` | `lkup_state` | `720_data_lkup_state.sql` |

### Adding / Editing an Institution

To replace a demo institution, edit its `name` (and optionally `institution_type_id`, `country_id`, `state_id`, `ror_id`) in place. To add one, append another `( ... )` block with a fresh `id`, keeping the column order shown above.

> ⚠️ **Note:** The seeded test user (`id = 3`) references `institution_id = 1` and `center_id = 1`. If you remove or repurpose institution `1` / Center `1`, update the test user block in the same file, or remove it if you don't want a seeded user.

### ROR IDs

The `ror_id` column holds a [Research Organization Registry](https://ror.org) URL identifying a real organization (e.g. `https://ror.org/00f54p054`). It is **optional** (`NULL` is valid). Only populate it with the genuine ROR URL for the real organization you are entering — never with a placeholder.

---

## Applying Your Changes

### Fresh Install

If you have not yet loaded the schema, just edit the SQL files in `canopy-cli/assets/db/postgres/init/` before running the schema deploy. The init scripts run in numeric order, so your edited `706` / `710` / `900` are picked up automatically. Follow Step 7a of the [Deployment Guide](./DEPLOYMENT_GUIDE.md).

### Existing Environment (Incremental)

`canopycli aws rds deploy-schema` only runs against a fresh database — it will **not** re-apply these files to an environment that is already initialized. For a running environment you must apply the changes by hand with `psql`:

```sql
-- Example: rename Center A everywhere
UPDATE public.lkup_center
   SET name = 'Coordinating Center'
 WHERE name = 'Center A';

UPDATE public.lkup_property_codelist_value
   SET value = 'Coordinating Center'
 WHERE property_codelist_id = 2 AND value = 'Center A';

-- Example: set a real institution + ROR id
UPDATE public.institution
   SET name = 'University of Example', ror_id = 'https://ror.org/xxxxxxxxx'
 WHERE id = 1;
```

See the **`canopy-deploy`** skill / runbook for connecting to RDS and the GRANT gotcha if you add brand-new tables (not needed for these data-only edits). Center/institution name changes are read by the services on the next request — no service restart is required for data-only updates.

---

## Out of Scope

This guide covers only Centers and affiliation institutions. The following also contain program-specific text but are part of the platform's data dictionary / branding, not seeded demo records, and are intentionally **not** changed here:

- `704_data_entity_property.sql`, `705_data_entity_property_display_setting.sql`, `707_data_entity_property_mta_mapping.sql` — entity-property descriptions and display labels.
- `726_data_lkup_core_variable_property_value.sql` — core-variable documentation text.
- `706` codelist `4` and other non-Center codelist values.
- The news / events / funding demo rows in `900_seed_test_data.sql` and their external URLs.

For frontend-side text and links (footer, Resource Center, etc.), see the [Frontend Customization Guide](./FRONTEND_CONTENT_CUSTOMIZATION_GUIDE.md).

---

**Last Updated:** June 2026
