# ANSIR Instrument Schedule Display

This is a copy of the code used to develop the web-based schedule interface for viewing ANSIR (Australian National Seismic Imaging Resource) instrument deployments and availability across multiple instrument categories. **This is a read-only display of current and planned deployments - it is not a booking system.**

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Configuration](#configuration)
4. [Adding Content](#adding-content)
5. [Customization](#customization)
6. [Dynamic Features](#dynamic-features)
7. [Maintenance](#maintenance)
8. [Troubleshooting](#troubleshooting)

## Overview

The schedule displays instrument availability and deployments across a 36-month period (2025-2027) using a split-table layout with accordion navigation. Each instrument type shows current deployments, maintenance status, and available instruments. Users can hover over deployment periods to see detailed experiment descriptions.

## Architecture

### File Structure
- `ANSIR_Schedule.html` - Main application file containing HTML, CSS, and JavaScript
- `README.md` - This documentation file

### Key Components

#### 1. Instrument Configuration (`instrumentPools` object)
Located in the JavaScript section (lines ~1000-1600), this object defines all instrument types and their properties:

```javascript
const instrumentPools = {
    instrumentKey: {
        total: 65,           // Total instruments in pool
        maintenance: 26,     // Instruments under maintenance
        experiments: [       // Array of table rows (availability + deployments)
            {
                name: "Available Instruments",  // Availability row (calculated automatically)
                isAvailability: true,
                deployments: [] // Empty - calculated by system
            },
            {
                name: "Experiment Name",        // Deployment row
                isAvailability: false,
                deployments: [
                    {
                        start: "2025-01",    // Start date (YYYY-MM)
                        end: "2025-09",      // End date (YYYY-MM)
                        count: 4,            // Number of instruments
                        description: "..."   // Optional description for tooltip
                    }
                ]
            }
        ]
    }
}
```

#### 2. HTML Structure
- **Categories**: Top-level accordions (Seismic Recorders, Seismometers, etc.)
- **Instruments**: Second-level accordions within each category
- **Tables**: Split-table layout with experiment names on left, calendar on right

#### 3. CSS Styling
- CSS variables for consistent theming (lines 7-60)
- Responsive design with mobile breakpoints
- Royal blue color scheme with hover effects

## Configuration

### Data Structure

#### Deployment Object
```javascript
{
    start: "2025-01",           // Start date (YYYY-MM)
    end: "2025-09",            // End date (YYYY-MM)
    count: 4,                  // Number of instruments
    description: "..."         // Optional description for tooltip
}
```

#### Experiment/Deployment Row Object
```javascript
{
    name: "Experiment Name",    // Display name for table row
    isAvailability: false,     // true for availability row, false for deployments
    deployments: [...]         // Array of deployment objects (empty for availability)
}
```

#### Availability Row Object
```javascript
{
    name: "Available Instruments",  // Always "Available Instruments"
    isAvailability: true,          // Must be true
    deployments: []                // Empty array - calculated automatically
}
```

### Modifying Instrument Counts

#### Total Instruments
Update the `total` property in the instrument configuration:

```javascript
instrumentKey: {
    total: 75,  // Changed from 65
    maintenance: 26,
    // ... rest of configuration
}
```

#### Maintenance Count
Update the `maintenance` property:

```javascript
instrumentKey: {
    total: 65,
    maintenance: 15,  // Changed from 26
    // ... rest of configuration
}
```

## Adding Content

### Adding New Instrument Types

#### 1. Add Instrument Configuration
Add a new entry to the `instrumentPools` object:

```javascript
newInstrumentKey: {
    total: 50,
    maintenance: 5,
    experiments: [
        {
            name: "Available Instruments",
            isAvailability: true,
            deployments: []
        },
        {
            name: "Your Experiment",
            deployments: [
                {
                    start: "2025-03",
                    end: "2025-08",
                    count: 10,
                    description: "Experiment description"
                }
            ]
        }
    ]
}
```

#### 2. Add HTML Structure
Add the instrument accordion item within the appropriate category:

```html
<div class="accordion-item">
    <div class="accordion-header" onclick="toggleAccordion(this)">
        <div>
            <div class="accordion-title">Instrument Display Name</div>
            <div class="accordion-subtitle">50 instruments (5 under maintenance)</div>
        </div>
        <div class="accordion-toggle">▼</div>
    </div>
    <div class="accordion-content">
        <div class="booking-calendar">
            <div class="calendar-flex-container">
                <div class="calendar-left">
                    <table class="booking-table-left" id="newInstrumentKey-table-left"></table>
                </div>
                <div class="calendar-right">
                    <div class="calendar-right-scroll">
                        <table class="booking-table-right" id="newInstrumentKey-table-right"></table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

**Important**: The table IDs must match the instrument key: `newInstrumentKey-table-left` and `newInstrumentKey-table-right`

#### 3. Update Category Mapping
Add the new instrument key to the appropriate category in `updateCategorySubtitles()` function:

```javascript
const categoryMappings = {
    'Seismic Recorders': ['terraSAWR', 'lpr200AUS', 'lpr200NZ', 'nanometricsCentaur', 'newInstrumentKey'],
    // ... other categories
};
```

### Adding New Categories

#### 1. Add Category HTML Structure
Add a new category accordion section:

```html
<div class="category-accordion">
    <div class="category-header" onclick="toggleCategory(this)">
        <div class="category-info">
            <div class="category-title">New Category Name</div>
            <div class="category-subtitle">X instrument types</div>
        </div>
        <div class="category-toggle">▼</div>
    </div>
    <div class="category-content">
        <div class="accordion-container">
            <!-- Instrument accordion items go here -->
        </div>
    </div>
</div>
```

#### 2. Add Category to JavaScript Mapping
Add the new category to the `categoryMappings` object in `updateCategorySubtitles()`:

```javascript
const categoryMappings = {
    'Seismic Recorders': ['terraSAWR', 'lpr200AUS', 'lpr200NZ', 'nanometricsCentaur'],
    'Seismometers': ['nanometrics120sAUS', 'nanometrics120sNZ', 'nanometricsH120s', 'nanometricsP120s', 'nanometrics20s'],
    'New Category Name': ['newInstrumentKey1', 'newInstrumentKey2'], // Add new category
    // ... other categories
};
```

#### 3. Update Category Count
Update the category subtitle to reflect the correct number of instrument types:

```html
<div class="category-subtitle">2 instrument types</div>
```

### Adding New Experiments

#### 1. Add to Existing Instrument
Add a new experiment object to the instrument's experiments array:

```javascript
{
    name: "New Experiment Name",
    deployments: [
        {
            start: "2025-06",
            end: "2026-02",
            count: 15,
            description: "Detailed experiment description for tooltip"
        }
    ]
}
```

#### 2. Add to Multiple Instruments
If an experiment uses multiple instrument types, add the same experiment configuration to each instrument's experiments array.

## Customization

### Styling Customization

#### Colors
Modify CSS variables in the `:root` section:

```css
:root {
    --primary-bg: #282572;        /* Main blue */
    --primary-hover: #1f1d5a;     /* Hover state */
    --available-bg: #d4edda;      /* Available instruments */
    --deployed-gradient: linear-gradient(135deg, #5bc0de 0%, #46b8da 100%);
    --today-color: #dc3545;       /* Today indicator */
}
```

#### Layout
- Month column width: `--month-width: 60px`
- Border radius: `--border-radius-*` variables
- Font sizes: `--font-size-*` variables

### Experiment Descriptions and Tooltips

#### Tooltip Functionality
When users hover over deployment periods (blue cells), a tooltip appears showing:

- **Experiment Name**: The full experiment title
- **Description**: Plain language summary of the experiment's purpose and scope
- **Date Range**: Start and end dates of the deployment
- **Instrument Count**: Number of instruments deployed

#### Adding Experiment Descriptions
Include descriptions in deployment objects for tooltip display:

```javascript
{
    name: "GA AusArray VIC",
    deployments: [
        {
            start: "2025-01",
            end: "2025-07",
            count: 58,
            description: "The Geoscience Australia AusArray Victoria project aims to enhance our understanding of Victoria's geological structures by deploying a dense network of passive seismic stations spaced approximately 50 kilometres apart over one year. Through advanced seismic imaging methods, including ambient noise tomography, body-wave tomography, and receiver functions, researchers will image the deep lithospheric structures beneath the region."
        }
    ]
}
```

#### Tooltip Styling
Tooltips are styled with:
- **Background**: Dark gray (`#333`)
- **Text**: White with good contrast
- **Positioning**: Follows mouse cursor, avoids screen edges
- **Max Width**: 400px with word wrapping
- **Z-Index**: High priority to appear above other elements

#### Tooltip Triggers
- **Hover**: Appears when mouse enters deployment cell
- **Move**: Follows mouse movement within cell
- **Leave**: Disappears when mouse exits cell
- **Edge Detection**: Automatically repositions to stay within viewport

### Today Indicator
The red "TODAY" line position is calculated automatically based on current date. Position offset can be adjusted in `addTodayIndicator()` function.

## Dynamic Features

### Dynamic Calculations

#### Current vs Upcoming Experiments
The system automatically categorizes experiments based on the current date:

- **Current Experiments**: Ongoing deployments that include the current month
- **Upcoming Experiments**: Deployments starting in future months
- **Past Experiments**: Deployments that have ended (not displayed in statistics)

**Calculation Logic:**
```javascript
const currentMonthIndex = parseMonth(currentMonthStr);
// Current: startIdx <= currentMonthIndex <= endIdx
// Upcoming: startIdx > currentMonthIndex
```

#### Availability Calculations
Available instruments are calculated dynamically:

1. **Total Available** = Total Instruments - Maintenance
2. **Currently Deployed** = Sum of all active deployments in current month
3. **Currently Available** = Total Available - Currently Deployed

**Algorithm:**
```javascript
function calculateAvailableInstruments(pool) {
    const totalAvailable = pool.total - pool.maintenance;
    const monthlyDeployments = new Array(36).fill(0);
    
    // Calculate deployments per month
    pool.experiments.forEach(exp => {
        if (!exp.isAvailability) {
            exp.deployments.forEach(dep => {
                const startIdx = parseMonth(dep.start);
                const endIdx = parseMonth(dep.end);
                for (let i = startIdx; i <= endIdx && i < 36; i++) {
                    monthlyDeployments[i] += dep.count;
                }
            });
        }
    });
    
    // Calculate available periods
    const availablePeriods = [];
    let i = 0;
    while (i < 36) {
        const available = totalAvailable - monthlyDeployments[i];
        let j = i;
        while (j < 36 && (totalAvailable - monthlyDeployments[j]) === available) {
            j++;
        }
        
        availablePeriods.push({
            start: `${startYear}-${startMonth}`,
            end: `${endYear}-${endMonth}`,
            count: available
        });
        
        i = j;
    }
    
    return availablePeriods;
}
```

**Example:**
- Total: 65 instruments
- Maintenance: 26 instruments
- Currently Deployed: 4 instruments (AHNA Auckland)
- **Currently Available**: 65 - 26 - 4 = 35 instruments

#### Category Statistics
Each category shows real-time statistics:

- **Instrument Types**: Count of different instrument models
- **Current Experiments**: Number of ongoing experiments
- **Upcoming Experiments**: Number of future experiments

#### Today Indicator Calculation
The red "TODAY" line position is calculated using:

```javascript
const today = new Date();
const todayYear = today.getFullYear();
const todayMonth = today.getMonth();
const baseIndex = (todayYear - startYear) * 12 + todayMonth;
const dayOfMonth = today.getDate();
const daysInMonth = new Date(todayYear, todayMonth + 1, 0).getDate();
const fraction = (dayOfMonth - 1) / daysInMonth;
const todayIndex = baseIndex + fraction;
```

**Positioning:**
- **Horizontal**: `left = todayIndex * (monthWidth + borderWidth)`
- **Vertical**: Spans the full height of the calendar body
- **Offset**: Automatically scrolls to show today ~4 months from left edge

### Real-Time Updates
The system automatically updates statistics based on current date:

- **Category Subtitles**: Show current vs upcoming experiment counts
- **Instrument Subtitles**: Display currently available instruments
- **Availability Rows**: Recalculate based on active deployments
- **Today Indicator**: Repositions based on current date

**Update Triggers:**
- Page load
- Window resize
- Accordion expansion/collapse

## Maintenance

### Browser Compatibility

- Modern browsers with ES6+ support
- CSS Grid and Flexbox support required
- Responsive design optimized for desktop and tablet

### Performance Considerations

- Tables are generated on page load
- Row height synchronization runs on scroll events
- Tooltip calculations are performed on hover
- Large instrument pools may impact initial load time

### Maintenance Checklist

- [ ] Update instrument totals and maintenance counts
- [ ] Add new experiments with proper date ranges
- [ ] Verify table IDs match instrument keys
- [ ] Test responsive design on different screen sizes
- [ ] Validate deployment date formats (YYYY-MM)
- [ ] Check tooltip descriptions for accuracy
- [ ] Update category mappings for new instruments and categories

## Troubleshooting

### Common Issues

1. **Tables not displaying**: Check table IDs match instrument keys
2. **Incorrect availability**: Verify deployment dates and counts
3. **Styling issues**: Check CSS variable definitions
4. **Tooltips not working**: Ensure deployment descriptions are properly formatted
5. **No booking functionality**: This is a read-only schedule display, not a booking system

### Debug Mode
Add `console.log()` statements in key functions to trace data flow and identify issues. 
