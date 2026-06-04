# Canopy Documentation

This repository contains comprehensive documentation for the Canopy platform.


## 📚 Contents

### Deployment & Operations

- **[Deployment Guide](./DEPLOYMENT_GUIDE.md)** - Complete step-by-step guide for deploying Canopy to AWS
  - Prerequisites and setup
  - AWS infrastructure deployment
  - Database schema setup
  - Application service deployment
  - Post-deployment verification
  - Troubleshooting

### Configuration & Customization

- **[Demo Data Setup](./DEMO_DATA_SETUP.md)** - Replacing the neutral placeholder Centers and affiliation institutions with your own
  - What the seeded demo data contains
  - The `710` ↔ `706` Center name-sync requirement
  - Adding, renaming, and removing Centers and institutions
  - Applying changes on a fresh install vs. an existing environment

### Optional & Reference

> ℹ️ The guides below are **not required** for a working deployment. They cover optional features, environment-specific tuning, and background reference. Skip them unless a section is relevant to your installation.

- **[Frontend Customization Guide](./FRONTEND_CONTENT_CUSTOMIZATION_GUIDE.md)** - Editing hardcoded frontend UI elements (footer links, Resource Center cards, social links)
- **[Google Analytics Setup](./GOOGLE_ANALYTICS_SETUP.md)** - Wiring up GA4 tracking in the UI (only if you want analytics)
- **[pg_cron Metrics Setup](./PGCRON_METRICS_SETUP.md)** - Scheduling automatic weekly hub-content metrics (Step 7b; without it, metrics are generated manually)
- **[Platform Limitations](./LIMITATIONS.md)** - Known constraints to be aware of when deploying and customizing

**Last Updated:** June 2026
