# Frontend Customization Guide

This guide explains how to customize various frontend UI elements in Canopy that are hardcoded in the codebase, including the Resource Center, homepage footer links, and other frontend-managed components.

## Table of Contents
- [Overview](#overview)
- [Homepage Footer Links](#homepage-footer-links)
- [Footer Social Media Links](#footer-social-media-links)
- [Resource Center](#resource-center)
  - [File Locations](#file-locations)
  - [Card Types and Categories](#card-types-and-categories)
  - [Card Structure](#card-structure)
  - [Editing Existing Cards](#editing-existing-cards)
  - [Adding New Cards to Existing Tabs](#adding-new-cards-to-existing-tabs)
  - [Creating a New Tab](#creating-a-new-tab)
  - [Button Types](#button-types)
  - [Examples](#examples)
  - [Quick Reference](#quick-reference)
---

## Overview

This guide covers customization of frontend UI elements that are hardcoded in the Canopy codebase. These elements require direct code changes rather than database or admin interface updates:

- **Homepage Footer Links** - Related Websites and Website Policies columns
- **Footer Social Media Links** - YouTube and Email/Newsletter icons in "Connect with Us" section
- **Resource Center** - Informational cards organized into tabbed categories
- Other frontend-managed components as needed

All changes require rebuilding and redeploying the frontend application to take effect.

---

## Homepage Footer Links

### Current Implementation

The homepage footer contains two columns of links - **Related Websites** and **Website Policies** - that are currently **hardcoded in the frontend codebase**. These links are not managed through a database or admin interface.

**Location:** `/datahub-ui-main/components/CoreLayout/Footer/Footer.jsx`

### Editing Footer Links

To customize the footer links for your deployment:

#### Step 1: Navigate to the Footer Component

File path: `/datahub-ui-main/components/CoreLayout/Footer/Footer.jsx`

#### Step 2: Edit the "Related Websites" Section

Located around lines 38-58 in the Footer component:

- Modify the `<a>` elements within the column with the "Related Websites" heading
- Update the `href` attribute with your desired URL
- Update the link text (between the `<a>` and `</a>` tags)
- Add or remove links by adding/removing `<a>` elements

#### Step 3: Edit the "Website Policies" Section

Located around lines 59-73 in the Footer component:

- Modify the `<a>` elements within the column with the "Website Policies" heading
- Follow the same steps as above to update URLs and link text

#### Step 4: Add External Link Icons (Optional)

To add an external link icon next to a link, wrap it in a `<div>` and add the `<ExternalLinkIcon />` component:

```jsx
<div>
    <a href="https://example.com" target="_blank" rel="noopener noreferrer">
        Link Text
    </a>
    <ExternalLinkIcon />
</div>
```

#### Step 5: Rebuild and Redeploy

- After making changes, rebuild the frontend application
- Redeploy to your environment for changes to take effect

### Example - Customizing a Footer Link

```jsx
// Before (default/placeholder):
<a href="/" target="_blank" rel="noopener noreferrer">
    Link
</a>

// After (customized):
<a href="https://www.nih.gov/" target="_blank" rel="noopener noreferrer">
    National Institutes of Health
</a>
```

**Note:** The "Site" column (first column) in the footer contains internal navigation links that use Next.js routing. These should generally remain unchanged as they point to core platform pages.

---

## Footer Social Media Links

### Current Implementation

The footer contains social media links in the "Connect with Us" section - **YouTube** and **Email/Newsletter** icons - that are currently **hardcoded in the frontend codebase**. These links are not managed through a database or admin interface.

**Location:** `/datahub-ui-main/components/CoreLayout/Footer/Footer.jsx`

### Editing Social Media Links

To customize the YouTube and email links in the footer:

#### Step 1: Navigate to the Footer Component

File path: `/datahub-ui-main/components/CoreLayout/Footer/Footer.jsx`

#### Step 2: Locate the Social Media Links Section

Look for the section with "Connect with Us:" text (around lines 86-100):

```jsx
<Row className={classes.socialText}>
    <div>Connect with Us:</div>
    <a href="/" target="_blank" rel="noopener noreferrer" aria-label="Youtube">
        <YoutubeIcon />
    </a>
    <a
        href="/"
        target="_blank"
        rel="noopener noreferrer"
        className={`px-2`}
        aria-label="Link to Sign Up for Newsletter"
    >
        <MailIcon />
    </a>
</Row>
```

#### Step 3: Update the YouTube Link

Modify the `href` attribute in the YouTube link (around line 88):

```jsx
// Before:
<a href="/" target="_blank" rel="noopener noreferrer" aria-label="Youtube">
    <YoutubeIcon />
</a>

// After:
<a href="https://www.youtube.com/@YourChannel" target="_blank" rel="noopener noreferrer" aria-label="Youtube">
    <YoutubeIcon />
</a>
```

#### Step 4: Update the Email/Newsletter Link

Modify the `href` attribute in the email/newsletter link (around lines 91-99):

**For a mailto link:**
```jsx
<a
    href="mailto:support@yourplatform.org"
    target="_blank"
    rel="noopener noreferrer"
    className={`px-2`}
    aria-label="Link to Sign Up for Newsletter"
>
    <MailIcon />
</a>
```

**For a newsletter signup page:**
```jsx
<a
    href="https://yourplatform.org/newsletter"
    target="_blank"
    rel="noopener noreferrer"
    className={`px-2`}
    aria-label="Link to Sign Up for Newsletter"
>
    <MailIcon />
</a>
```

#### Step 5: Rebuild and Redeploy

- After making changes, rebuild the frontend application
- Redeploy to your environment for changes to take effect

### Complete Example

```jsx
<Row className={classes.socialText}>
    <div>Connect with Us:</div>
    <a href="https://www.youtube.com/@YourResearchChannel" target="_blank" rel="noopener noreferrer" aria-label="Youtube">
        <YoutubeIcon />
    </a>
    <a
        href="mailto:contact@yourresearchplatform.org"
        target="_blank"
        rel="noopener noreferrer"
        className={`px-2`}
        aria-label="Email Us"
    >
        <MailIcon />
    </a>
</Row>
```

**Note:** You can also update the `aria-label` attribute to provide a more descriptive label for accessibility purposes.

---

## Resource Center

### Overview

The Resource Center displays informational cards organized into tabbed categories. Each tab contains cards relevant to different user groups or content types. The Canopy includes four tabs:

1. **All** - Displays all cards from all categories in one view (automatically generated)
2. **General** - Platform-wide information (News, Events, FAQ, etc.)
3. **For Researchers** - Resources specific to data consumers
4. **For Submitters** - Resources specific to data contributors

Each category tab (General, For Researchers, For Submitters) has its own component file, color scheme, and icon. The "All" tab automatically aggregates cards from all categories and does not require a separate file.

### File Locations

#### Card Component Files

Each category tab has a dedicated card component file:

| Tab | File Location | Type Value |
|-----|---------------|------------|
| All | *(No separate file - auto-generated from all categories)* | `'all'` |
| General | `datahub-ui-main/views/ResourceCenter/Components/GeneralCards.jsx` | `'general'` | 
| For Researchers | `datahub-ui-main/views/ResourceCenter/Components/ForResearchersCards.jsx` | `'forResearchers'` | 
| For Submitters | `datahub-ui-main/views/ResourceCenter/Components/ForSubmittersCards.jsx` | `'forSubmitters'` | 

**Note:** The "All" tab automatically displays cards from all category tabs combined. You do not need to edit or create a separate file for the "All" tab - it automatically updates when you add or modify cards in any category tab.

#### Main Resource Center File

The main component that coordinates all tabs:
```
datahub-ui-main/views/ResourceCenter/ResourceCenter.jsx
```

#### Styling

```
datahub-ui-main/views/ResourceCenter/ResourceCenter.module.scss
```

---

### Card Types and Categories

#### The "All" Tab

The "All" tab is a special tab that automatically aggregates and displays cards from all other category tabs. It is defined in the main `ResourceCenter.jsx` file:

```javascript
const allCards = (router, baseUrl) => {
    return [
        ...generalCards(router, baseUrl, restGet),
        ...forResearchersCards(router, baseUrl, restGet),
        ...forSubmittersCards(router, baseUrl, restGet),
    ];
};
```

#### Card Structure

Each card is a JavaScript object defined in its respective card component file (`GeneralCards.jsx`, `ForResearchersCards.jsx`, or `ForSubmittersCards.jsx`).

**Card Object Structure:**

```javascript
{
    title: 'Card Title',           // String - Displayed as the card header
    type: 'general',                // String - Card category type (must match the tab)
    children: (                     // JSX - Card body content
        <>
            <p>Your content here</p>
        </>
    ),
    footer: (                       // JSX - Card footer with buttons/actions
        <span className={classes.resourceCardFooter}>
            {/* Buttons or links */}
        </span>
    ),
}
```

#### How Category Tabs Are Defined

Each category tab is imported and used in the main `ResourceCenter.jsx` file. Here's how they are structured:

**Import Statements** (at the top of ResourceCenter.jsx):
```javascript
import { generalCards } from './Components/GeneralCards';
import { forResearchersCards } from './Components/ForResearchersCards';
import { forSubmittersCards } from './Components/ForSubmittersCards';
```

**Tab Rendering** (in the ResourceCenter.jsx):
```javascript
<Tabs
    id="controlled-resources"
    activeKey={key}
    onSelect={(k) => {
        setKey(k);
        setActiveColor(categoryColors[k]);
    }}
>
    <Tab eventKey="all" title={tabLabels.all}>
        <Row>{renderCards(allCards, router, baseUrl, restGet)}</Row>
    </Tab>
    <Tab eventKey="general" title={tabLabels.general}>
        <Row>{renderCards(generalCards, router, baseUrl, restGet)}</Row>
    </Tab>
    <Tab eventKey="forResearchers" title={tabLabels.forResearchers}>
        <Row>{renderCards(forResearchersCards, router, baseUrl, restGet)}</Row>
    </Tab>
    <Tab eventKey="forSubmitters" title={tabLabels.forSubmitters}>
        <Row>{renderCards(forSubmittersCards, router, baseUrl, restGet)}</Row>
    </Tab>
</Tabs>
```

---

### Editing Existing Cards

#### Step 1: Identify the Tab

Determine which tab contains the card you want to edit:
- **General tab** → `GeneralCards.jsx`
- **For Researchers tab** → `ForResearchersCards.jsx`
- **For Submitters tab** → `ForSubmittersCards.jsx`

#### Step 2: Open the File

Navigate to and open the appropriate file:
```
datahub-ui-main/views/ResourceCenter/Components/[CardFileName].jsx
```

#### Step 3: Locate the Card

Find the card you want to edit in the exported function. Cards are stored in an array (typically starting around line 21).

#### Step 4: Modify the Content

**Example - Editing the "News" card in General tab:**

```javascript
{
    title: 'News',                  // Change the title
    type: 'general',                // Keep the type matching the file
    children: (
        <>
            <p>
                Stay updated with platform announcements,
                feature releases, and community highlights.
            </p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/news">     // Change the link destination
                <Button 
                    className={moreButtonClasses} 
                    label="View News"   // Change button label
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
        </span>
    ),
},
```
---

### Adding New Cards to Existing Tabs

#### Step 1: Choose the Target Tab

Decide which tab should contain your new card:
- **General** - Platform information, news, events, FAQs
- **For Researchers** - Data access, analysis tools, tutorials
- **For Submitters** - Submission guides, data standards, templates

#### Step 2: Open the Appropriate File

Open the corresponding file:
```
datahub-ui-main/views/ResourceCenter/Components/[CardFileName].jsx
```

#### Step 3: Add the Card Object

Add a new card object to the array. Use the template matching your chosen tab:

**For General Cards:**
```javascript
{
    title: 'Your New Card Title',
    type: 'general',
    children: (
        <>
            <p>Your card description goes here.</p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/your-route">
                <Button 
                    className={moreButtonClasses} 
                    label="View Page" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
        </span>
    ),
},
```

**For Researchers Cards:**
```javascript
{
    title: 'Your New Card Title',
    type: 'forResearchers',        // Different type
    children: (
        <>
            <p>Your card description goes here.</p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/your-route">
                <Button 
                    className={moreButtonClasses}  // Uses teal color
                    label="View Page" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
        </span>
    ),
},
```

**For Submitters Cards:**
```javascript
{
    title: 'Your New Card Title',
    type: 'forSubmitters',         // Different type
    children: (
        <>
            <p>Your card description goes here.</p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/your-route">
                <Button 
                    className={moreButtonClasses}  // Uses dark blue color
                    label="View Page" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
        </span>
    ),
},
```

#### Step 4: Position the Card

Decide where in the array to place your card. Cards display in the order they appear. Add your card object with a comma after the previous card.

#### Step 5: Complete Example

Adding a "Data Access Request" card to the For Researchers tab:

```javascript
// In ForResearchersCards.jsx
{
    title: 'User Tutorial',
    type: 'forResearchers',
    children: (
        <>
            <p>
                Learn how to navigate the platform, search for datasets,
                and use analytical tools effectively.
            </p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/tutorial">
                <Button className={moreButtonClasses} label="View Page" variant="primary" size="auto" rounded="lite" />
            </Link>
        </span>
    ),
},
// NEW CARD ADDED HERE
{
    title: 'Data Access Request',
    type: 'forResearchers',
    children: (
        <>
            <p>
                Submit a data access request to gain permission for
                controlled-access datasets and restricted data files.
            </p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/dataAccess">
                <Button 
                    className={moreButtonClasses} 
                    label="Request Access" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
        </span>
    ),
},
```

---

### Creating a New Tab

To add an entirely new tab to the Resource Center, follow these steps:

#### Step 1: Create a New Card Component File

1. Navigate to `datahub-ui-main/views/ResourceCenter/Components/`
2. Create a new file (e.g., `ForPartnersCards.jsx`)
3. Use this template:

```javascript
import React from 'react';
import PropTypes from 'prop-types';
import Link from 'next/link';
import Button from '../../../components/Button/Button';
import DownloadIcon from '../../../components/Images/svg/DownloadIcon';
import classes from '../ResourceCenter.module.scss';
import { GET_RESOURCE_CENTER_BUCKET } from '../../../constants/apiRoutes';
import { downloadLink } from '../../../lib/pageHelpers/downloadLink';
import { sendGAEvent } from '@next/third-parties/google';

/**
 * For Partners Resource Cards
 * @property {Object} router - Next router to be used for button handleClick functions
 * @returns {Array} Array of Objects for Card data
 */

// Choose a color: navyBlue, teal, darkBlue, green, or add a new color in SCSS
const moreButtonClasses = `${classes.moreButton} ${classes.green}`;
const downloadButtonClasses = `${classes.downloadButton} ${classes.green}`;

export const forPartnersCards = (router, baseUrl, restGet) => {
    return [
        {
            title: 'Your First Card',
            type: 'forPartners',
            children: (
                <>
                    <p>
                        Description of your card content goes here.
                    </p>
                </>
            ),
            footer: (
                <span className={classes.resourceCardFooter}>
                    <Link href="/partners">
                        <Button 
                            className={moreButtonClasses} 
                            label="View Page" 
                            variant="primary" 
                            size="auto" 
                            rounded="lite" 
                        />
                    </Link>
                </span>
            ),
        },
        // Add more cards here...
    ];
};

forPartnersCards.PropTypes = {
    router: PropTypes.object,
};
```

#### Step 2: Update ResourceCenter.jsx

Open `datahub-ui-main/views/ResourceCenter/ResourceCenter.jsx` and make the following changes:

**2.1: Import your new card component** (around line 14-17):
```javascript
import { generalCards } from './Components/GeneralCards';
import { forResearchersCards } from './Components/ForResearchersCards';
import { forSubmittersCards } from './Components/ForSubmittersCards';
import { forPartnersCards } from './Components/ForPartnersCards'; // ADD THIS
```

**2.2: Add to allCards function** (around line 34-41):

This step is important - it ensures your new tab's cards appear in the "All" tab view.

```javascript
const allCards = (router, baseUrl) => {
    return [
        ...generalCards(router, baseUrl, restGet),
        ...forResearchersCards(router, baseUrl, restGet),
        ...forSubmittersCards(router, baseUrl, restGet),
        ...forPartnersCards(router, baseUrl, restGet),  // ADD THIS
    ];
};
```

**2.3: Add category color** (around line 74):
```javascript
const categoryColors = { 
    all: 'black', 
    general: 'navyBlue', 
    forResearchers: 'teal', 
    forSubmitters: 'darkBlue',
    forPartners: 'green'  // ADD THIS
};
```

**2.4: Add category header image** (around line 75-80):
```javascript
const categoryHeaders = {
    general: '/images/navy_blue_sml_header.png',
    forResearchers: '/images/light_blue_sml_header.png',
    forSubmitters: '/images/dark_blue_sml_header.png',
    forPartners: '/images/green_sml_header.png',  // ADD THIS
};
```

**2.5: Add color case in switch statement** (around line 99-118):
```javascript
switch (activeColor) {
    case 'black':
        activeColorClass += ` ${classes.black}`;
        break;
    case 'navyBlue':
        activeColorClass += ` ${classes.navyBlue}`;
        break;
    case 'teal':
        activeColorClass += ` ${classes.teal}`;
        break;
    case 'darkBlue':
        activeColorClass += ` ${classes.darkBlue}`;
        break;
    case 'green':
        activeColorClass += ` ${classes.green}`;
        break;
    // ADD YOUR COLOR CASE IF USING A NEW COLOR
    default:
        activeColorClass += ` ${classes.navyBlue}`;
        break;
}
```

**2.6: Add tab label** (around line 148-174):
```javascript
import { Box, Archive, Clipboard2Pulse, CloudUpload, Link45deg, People } from 'react-bootstrap-icons'; // Import icon

const tabLabels = {
    all: (
        <span>
            <Box /> All
        </span>
    ),
    general: (
        <span>
            <Archive /> General
        </span>
    ),
    forResearchers: (
        <span>
            <Clipboard2Pulse /> For Researchers
        </span>
    ),
    forSubmitters: (
        <span>
            <CloudUpload /> For Submitters
        </span>
    ),
    forPartners: (  // ADD THIS
        <span>
            <People /> For Partners
        </span>
    ),
};
```

**2.7: Add icon definition** (around line 176-182):
```javascript
const icons = {
    all: <Box className={classes.titleIcon} />,
    general: <Archive className={classes.titleIcon} />,
    forResearchers: <Clipboard2Pulse className={classes.titleIcon} />,
    forSubmitters: <CloudUpload className={classes.titleIcon} />,
    forPartners: <People className={classes.titleIcon} />,  // ADD THIS
};
```

**2.8: Add the Tab component** (around line 210-239):
```javascript
<Tabs
    id="controlled-resources"
    activeKey={key}
    onSelect={(k) => {
        setKey(k);
        setActiveColor(categoryColors[k]);
    }}
    className={`mb-3 ${classes.resourceTabs} ${activeColorClass} narrowTextBackground`}
>
    <Tab eventKey="all" title={tabLabels.all}>
        {searchBar}
        <Row className={classes.Row}>{renderCards(allCards, router, baseUrl, restGet)}</Row>
    </Tab>
    <Tab eventKey="general" title={tabLabels.general}>
        {searchBar}
        <Row className={classes.Row}>{renderCards(generalCards, router, baseUrl, restGet)}</Row>
    </Tab>
    <Tab eventKey="forResearchers" title={tabLabels.forResearchers}>
        {searchBar}
        <Row className={classes.Row}>{renderCards(forResearchersCards, router, baseUrl, restGet)}</Row>
    </Tab>
    <Tab eventKey="forSubmitters" title={tabLabels.forSubmitters}>
        {searchBar}
        <Row className={classes.Row}>{renderCards(forSubmittersCards, router, baseUrl, restGet)}</Row>
    </Tab>
    {/* ADD YOUR NEW TAB HERE */}
    <Tab eventKey="forPartners" title={tabLabels.forPartners}>
        {searchBar}
        <Row className={classes.Row}>{renderCards(forPartnersCards, router, baseUrl, restGet)}</Row>
    </Tab>
</Tabs>
```

#### Step 3: (Optional) Add Custom Color

If you want a new color scheme not already available, add it to the SCSS file:

Open `datahub-ui-main/views/ResourceCenter/ResourceCenter.module.scss` and add your color classes following the existing pattern.

---

### Button Types

#### 1. Navigation Button (Internal Link)

Links to another page within the application:

```javascript
<Link href="/your-page">
    <Button 
        className={moreButtonClasses} 
        label="View Page" 
        variant="primary" 
        size="auto" 
        rounded="lite" 
    />
</Link>
```

#### 2. Download Button

Triggers a file download:

```javascript
<Button
    className={downloadButtonClasses}
    label="PDF (433KB)"
    iconLeft={<DownloadIcon />}
    variant="primary"
    size="auto"
    rounded="lite"
    handleClick={async () => {
        // Download logic here
        await downloadLink(
            `${baseUrl}${GET_RESOURCE_CENTER_BUCKET}your-file.pdf`,
            'your-file.pdf'
        );
    }}
/>
```

#### 3. Multiple Buttons

Cards can have both navigation and download buttons:

```javascript
footer: (
    <span className={classes.resourceCardFooter}>
        <Link href="/faq">
            <Button 
                className={moreButtonClasses} 
                label="View Page" 
                variant="primary" 
                size="auto" 
                rounded="lite" 
            />
        </Link>
        <Button
            className={downloadButtonClasses}
            label="PDF (522KB)"
            iconLeft={<DownloadIcon />}
            variant="primary"
            size="auto"
            rounded="lite"
            handleClick={async () => {
                await downloadLink(
                    `${baseUrl}${GET_RESOURCE_CENTER_BUCKET}faq.pdf`,
                    'faq.pdf'
                );
            }}
        />
    </span>
)
```

#### 4. Right-Aligned Button

For single download button aligned to the right:

```javascript
footer: (
    <span className={classes.resourceCardFooter}>
        <div className={classes.footerEnd}>
            <Button
                className={downloadButtonClasses}
                label="XLSX (96KB)"
                iconLeft={<DownloadIcon />}
                variant="primary"
                size="auto"
                rounded="lite"
                handleClick={async () => {
                    // Download logic
                }}
            />
        </div>
    </span>
)
```

---

### Examples

#### Example 1: Simple Information Card (General Tab)

```javascript
// In GeneralCards.jsx
{
    title: 'Platform Guidelines',
    type: 'general',
    children: (
        <>
            <p>
                Review our platform usage guidelines and best 
                practices for data submission and access.
            </p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/guidelines">
                <Button 
                    className={moreButtonClasses} 
                    label="Read Guidelines" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
        </span>
    ),
},
```

#### Example 2: Card with Download Only (For Submitters Tab)

```javascript
// In ForSubmittersCards.jsx
{
    title: 'Data Submission Template',
    type: 'forSubmitters',
    children: (
        <>
            <p>
                Download the standardized template for submitting 
                your research data to ensure compliance with 
                platform requirements.
            </p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <div className={classes.footerEnd}>
                <Button
                    className={downloadButtonClasses}
                    label="XLSX (1.5MB)"
                    iconLeft={<DownloadIcon />}
                    variant="primary"
                    size="auto"
                    rounded="lite"
                    handleClick={async () => {
                        await downloadLink(
                            `${baseUrl}${GET_RESOURCE_CENTER_BUCKET}submission-template.xlsx`,
                            'submission-template.xlsx'
                        );
                        sendGAEvent('event', 'download', {
                            value: 'submission-template'
                        });
                    }}
                />
            </div>
        </span>
    ),
},
```

#### Example 3: Card with Both Buttons (For Researchers Tab)

```javascript
// In ForResearchersCards.jsx
{
    title: 'Analysis Tutorials',
    type: 'forResearchers',
    children: (
        <>
            <p>
                Learn how to use our analytical tools including 
                Jupyter notebooks, R, and Python for data analysis.
            </p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/tutorials">
                <Button 
                    className={moreButtonClasses} 
                    label="View Online" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
            <Button
                className={downloadButtonClasses}
                label="PDF (3.2MB)"
                iconLeft={<DownloadIcon />}
                variant="primary"
                size="auto"
                rounded="lite"
                handleClick={async () => {
                    await downloadLink(
                        `${baseUrl}${GET_RESOURCE_CENTER_BUCKET}analysis-tutorial.pdf`,
                        'analysis-tutorial.pdf'
                    );
                }}
            />
        </span>
    ),
},
```

#### Example 4: Card with Rich Content (General Tab)

```javascript
// In GeneralCards.jsx
{
    title: 'Getting Started',
    type: 'general',
    children: (
        <>
            <p>
                New to the platform? Follow these steps:
            </p>
            <ul>
                <li>Create an account and complete registration</li>
                <li>Explore available datasets</li>
                <li>Request data access</li>
                <li>Begin your analysis</li>
            </ul>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <Link href="/gettingStarted">
                <Button 
                    className={moreButtonClasses} 
                    label="Start Here" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </Link>
        </span>
    ),
},
```

#### Example 5: Card with External Link (For Researchers Tab)

```javascript
// In ForResearchersCards.jsx
{
    title: 'External Resources',
    type: 'forResearchers',
    children: (
        <>
            <p>
                Access additional research resources and 
                documentation from external partners and repositories.
            </p>
        </>
    ),
    footer: (
        <span className={classes.resourceCardFooter}>
            <a href="https://example.org/resources" target="_blank" rel="noreferrer">
                <Button 
                    className={moreButtonClasses} 
                    label="Visit Site" 
                    variant="primary" 
                    size="auto" 
                    rounded="lite" 
                />
            </a>
        </span>
    ),
},
```

---

### Quick Reference

#### Card Type Summary

| Tab | File | Type Value | Button Color | Icon |
|-----|------|------------|--------------|------|
| All | *(Auto-generated)* | `'all'` | Black | Box |
| General | `GeneralCards.jsx` | `'general'` | Navy Blue | Archive |
| For Researchers | `ForResearchersCards.jsx` | `'forResearchers'` | Teal | Clipboard2Pulse |
| For Submitters | `ForSubmittersCards.jsx` | `'forSubmitters'` | Dark Blue | CloudUpload |

**Important:** The "All" tab is automatically populated by combining cards from all category tabs. When you add or edit cards in any category, they will automatically appear in the "All" tab.

#### Common Tasks Quick Links

- **Edit existing card:** Find the card in its tab's file → Modify title/content/footer → Save → Build
- **Add new card:** Choose tab → Open file → Add card object → Save → Build
- **Create new tab:** Create card file → Update ResourceCenter.jsx (8 places) → Build → Test
- **Change button:** Modify `label`, `href`, or `handleClick` in footer section
- **Add download:** Add Button with `iconLeft={<DownloadIcon />}` and `handleClick` with `downloadLink`

---

## Related Documentation

- [Platform Limitations](./LIMITATIONS.md) - Understanding platform constraints and architectural decisions
- [Deployment Guide](./DEPLOYMENT_GUIDE.md) - Full platform deployment instructions
- [README](../README.md) - Platform overview and setup

---

**Last Updated:** January 2026  
**Version:** 1.0

