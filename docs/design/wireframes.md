# Design System & UI/UX Guidelines

[View Complete Design on Figma](https://www.figma.com/design/5w46S5myWDgoaayVKgvy7s/Mailisha?node-id=1730-221&t=5uNbFqzVjUklvtPO-0)

## Heliks Team - Color, Design & Style Guide

### Brand Identity
The Heliks brand represents reliability, transparency, and innovation in mineral supply chain traceability.

### Color Palette

| Color Name | HEX Code | Usage | Purpose |
|---|---|---|---|
| Primary Navy | `#262262` | Headers, primary buttons, navigation | Brand identity and authority |
| Primary Teal | `#0BC08F` | Accent buttons, active states, highlights | Innovation and growth |
| Teal Dark | `#09A77C` | Hover states | Interactive feedback |
| Teal Soft | `#CDF3EA` | Backgrounds, highlights | Subtle emphasis |
| Navy Hover | `#332D7A` | Hover states for primary elements | Interactive states |
| White | `#FFFFFF` | Backgrounds, cards | Clean space |
| Text Primary | `#111111` | Body text | Readability |
| Border/Glass | `#E3E2F0` | Borders, dividers | Separation |

### Typography

| Font | Weights | Usage |
|---|---|---|
| Poppins | 400, 500, 600, 700 | Primary font for all UI elements, headings, and body text |
| JetBrains Mono | 400, 500 | Code snippets, technical content, and data displays |

### Logo Design
- **Primary Logo:** RICA/Heliks logo in navy (#262262) for light mode
- **Inverse Logo:** RICA logo in white for dark mode on navy backgrounds
- **Minimum Size:** 32px height
- **Clear Space:** Equal to the logo height on all sides

### Icons
- **Style:** Lucide React icon library - clean, modern, consistent stroke-based icons
- **Size:** 16px, 20px, 24px standard sizes
- **Color:** Inherit from text color or use brand colors for emphasis

### Buttons Design

| Button Type | Background | Text Color | Usage |
|---|---|---|---|
| Primary | Gradient (#262262 to #1a194a) | White | Main actions (Login, Get Started) |
| Secondary/Accent | #0BC08F | White | Secondary actions, CTAs |
| Outline | White with #262262 border | #262262 | Alternative actions |
| Ghost | Transparent | Inherit | Tertiary actions |

## Product Design - Web App

### Design Principles (8 Principles of Design)
1. **Emphasis** - Highlight critical actions and information through color, size, and placement
2. **Balance** - Distribute visual weight evenly across layouts
3. **Unity/Harmony** - Maintain consistency in colors, typography, and spacing
4. **Rhythm/Pattern** - Use repeating elements to create visual flow
5. **Proportion/Scale** - Size elements appropriately based on importance
6. **Contrast** - Ensure readability through proper color contrast
7. **Hierarchy** - Guide user attention through size, weight, and color
8. **White Space** - Use breathing room for clarity and focus

### 8 Elements of Design
1. **Line** - Used for borders, dividers, and navigation indicators
2. **Shape** - Cards, buttons, and containers with consistent border-radius
3. **Form/Space** - Layout structure using grids and flexbox
4. **Texture** - Subtle shadows and gradients for depth
5. **Color** - Brand palette applied consistently
6. **Value/Tone** - Light and dark mode support
7. **Size/Scale** - Responsive sizing across breakpoints
8. **Typography** - Poppins font family throughout

### Product Web App Wireframes - Role-Based Designs

#### System Administrator Interface
- **Dashboard:** Company and user management overview
- **Company Management:** Company registration and listing
- **User Management:** User creation, role assignment, enable/disable functionality
- **Account Settings:** Admin profile management

#### Operations Manager Interface
- **Dashboard:** Batch overview and status tracking
- **Batch Management:** Create, view, and track mineral batches
- **Processing Stages:** Stage progression and history
- **Settings:** User account settings

#### Supply Chain Manager Interface
- **Dashboard:** Value analysis overview
- **Value Calculations:** LME pricing integration and calculations
- **Scenario Planning:** What-if analysis with adjustable parameters
- **Reports:** Value tables and analysis

### Informational Website Design
- **Hero Section:** RICA platform introduction with clear CTAs
- **About Section:** Platform purpose and value proposition
- **Product Section:** Features, how it works (3-step process), and impact
- **Team Section:** Heliks team showcase
- **Contact Section:** Contact information and links
- **Responsive Design:** Mobile-first approach with adaptive layouts

## Testing Strategy

### Frontend Testing

#### Playwright (E2E Testing)
Playwright is used for end-to-end testing to simulate real user interactions across the application.

- **Purpose:** Test complete user flows from login to task completion
- **Coverage:** Authentication, role-based access, batch management, value calculations
- **Features:** Cross-browser testing, API mocking, screenshot comparisons, trace debugging
- **Configuration:** `playwright.config.ts` with Chromium browser
- **Test Location:** `tests/e2e/`

#### Cypress (Component & E2E Testing)
Cypress provides an alternative testing approach for component testing and interactive debugging.

- **Purpose:** Component testing and interactive end-to-end testing with real-time debugging
- **Features:** Time-travel debugging, automatic waiting, network stubbing
- **Use Case:** Testing individual components in isolation and complex user interactions
- **Integration:** Works alongside Playwright for comprehensive test coverage

Both testing frameworks ensure the application works correctly across different scenarios, user roles, and edge cases.
