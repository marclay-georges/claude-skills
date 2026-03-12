---
name: browser-automation
description: Expert browser control and web scraping using accessibility trees, element references, and intelligent tool selection. Use when the user needs to interact with websites, extract data, automate web workflows, fill forms, or scrape content. Combines Claude in Chrome's flexibility with Playwright's speed based on task characteristics.
---

---
name: browser-automation
description: Expert browser control and web scraping using accessibility trees, element references, and intelligent tool selection. Use when the user needs to interact with websites, extract data, automate web workflows, fill forms, or scrape content. Combines Claude in Chrome's flexibility with Playwright's speed based on task characteristics.
---

# Browser Automation Expert

This skill helps you understand browser automation tools and techniques, focusing on accessibility trees and element references as your primary navigation method.

## Core Insight: Structure Before Vision

The accessibility tree reveals page structure in a way that's both machine-readable and human-understandable. When you use `read_page` first, you get:
- Element references (ref_ids) that are stable and reliable for interaction
- Understanding of the page's semantic structure
- Text content and interactive elements without visual rendering
- A foundation for making informed decisions about what to do next

Screenshots are valuable for verification and spatial understanding, but the accessibility tree gives you actionable references. Think of it as reading the blueprint before decorating the room.

## Tool Ecosystems: Characteristics and Trade-offs

You have two complementary approaches available:

### Claude in Chrome
**Character**: Flexible explorer that handles ambiguity well
- Works through accessibility trees and natural language element finding
- More resilient when page structure is messy or unpredictable
- Each action requires more tool calls but adapts to what it encounters
- Better for discovery, exploration, and dealing with real-world chaos

### Playwright
**Character**: Fast executor that values clarity
- Direct programmatic control with explicit selectors
- Efficient when you know exactly what you're doing
- Strict mode can fail on ambiguous selectors (which is common on modern sites)
- Better for speed, loops, and repeatable workflows

**Neither is "better"** - they excel in different contexts. The question is always: what does this specific problem need? You might start with one and shift to the other based on what you discover. Chrome for figuring things out, Playwright for executing at scale. Or vice versa if you already know the territory.

## Understanding Page Structure

When approaching a new page, `read_page` gives you the lay of the land. You're seeing:
- What elements exist and how they're organized
- Which elements are interactive (buttons, links, form fields)
- Text content and labels that indicate purpose
- Reference IDs (ref_ids) that let you interact with specific elements

The depth parameter controls how deep into the tree you go. Deeper gives more detail but more output. If you're getting truncated output or need to focus, you can:
- Lower the depth to see less (8-10 for overview)
- Use a ref_id to focus on a specific part of the page
- Filter to "interactive" to see only actionable elements

### Finding Elements

You have several ways to locate what you need:

**read_page with filters** gives you the full structure - you parse it yourself to find what matters. Good when you want to understand relationships between elements or see everything at once.

**find tool** uses natural language - "search button", "price filter", "login form". It returns up to 20 matches with their refs. Good when you know conceptually what you're looking for but not where it is structurally.

**Screenshots** show you spatial layout and visual design. They help verify what you're seeing in the accessibility tree matches what a human would see. Less useful for interaction since you can't directly reference visual elements - you'd need to translate coordinates.

The pattern that tends to work: read the structure first, identify what you need, then interact using references. Screenshots confirm rather than drive.

## Interaction Approaches

### Form Filling: Batch vs Sequential

`form_input` lets you fill multiple fields at once using their refs. It's efficient and handles different input types (text, dropdowns, checkboxes) uniformly. You gather all the refs from `read_page`, map your data to them, and fill everything in one operation.

The alternative is clicking into each field and typing, which gives you more control over timing and can trigger field-level validation or JavaScript handlers that `form_input` might bypass. Some forms expect this sequential interaction.

The trade-off: speed and simplicity vs control and compatibility with complex form logic. Most forms work fine with `form_input`. When they don't (usually you'll see validation failures or the form won't submit), fall back to the click-and-type approach.

### Navigation: URLs vs Clicks

The `navigate` tool is the most reliable way to go somewhere - you're directly setting the URL. It works even when link elements are complex or when the page uses JavaScript navigation.

Clicking links using refs from `read_page` is more like how a human navigates. Sometimes that matters - some sites track click behavior. But it's also more fragile if the link is inside a complex interactive element.

For back/forward, the navigate tool handles it directly. For JavaScript-heavy single-page apps, sometimes you need to trigger the navigation through interaction rather than URL changes.

Think about what kind of navigation the site expects and how robust you need to be.

### Interacting with Elements

When you have a ref from `read_page` or `find`, you can pass it directly to the computer tool or form_input. This is usually the smoothest path - you're referencing exactly what you identified in the structure.

If you don't have a ref (or it's not working), coordinates from a screenshot work but are more brittle - they break if window size changes or if the page layout shifts.

The general flow: identify element → get ref → interact using ref → verify the result. That last step matters - `read_page` after an interaction tells you if the page state changed as expected.

## Data Extraction: Reading vs Scraping

When you need data from a page, you're essentially translating from the page's structure into a format you can work with.

### Understanding Data Layout First

Before extracting, look at how the data is organized:
- Is it in a semantic table? Lists? Cards in a grid?
- Does it repeat in a predictable pattern?
- Is there pagination, infinite scroll, or "load more" patterns?
- Is the content in the DOM or loaded via JavaScript/API calls?

This understanding shapes your extraction approach.

### Extraction Methods: Characteristics

**Accessibility Tree Parsing**
You already have the structure from `read_page` - you're just pulling out the pieces you need. Works well when content is in the DOM and has semantic meaning. You're pattern-matching against the tree structure to find repeating elements.

**get_page_text**
Gives you clean text content prioritized for articles and main content. Fast and simple, but you lose structural information. Good for reading, less good for structured data extraction. Think of it as "reader mode" - great for prose, limiting for tabular data.

**JavaScript Extraction**
The `javascript_tool` lets you query the DOM directly and return structured data as JSON. More powerful for complex extractions - you can traverse relationships, aggregate data, access page variables. You're writing the extraction logic yourself, which gives you precision but requires understanding the DOM structure.

Example: Extracting all products with their names and prices from cards - you'd query all product elements, map over them, extract specific fields, return as an array.

**Network Inspection**
Many modern sites load data through API calls rather than embedding it in HTML. `read_network_requests` shows you these calls. Sometimes you can extract the data from API responses directly, bypassing all the DOM manipulation. It's cleaner when it works, but requires the data to be in network requests rather than pre-rendered.

The trade-off is between simplicity (text extraction) and power (JavaScript/network inspection). Start simple, add complexity as needed.

### Pagination Patterns

Websites handle multi-page content in different ways:

**Click-based navigation**: "Next" buttons or numbered page links. You extract from the current page, find the next button's ref, click it, wait for the new content to load, repeat. The challenge is knowing when you've reached the end - usually the "next" button disappears or becomes disabled.

**URL-based pagination**: The page number is in the URL (?page=2, /page/3, etc.). You can navigate directly to each page URL, which is often more reliable than clicking - no waiting for JavaScript, no risk of missing a click. The pattern is usually obvious once you see a few URLs.

**Infinite scroll**: New content loads as you scroll down. You need to scroll, wait for loading, check for new content, repeat. Detecting when there's truly no more content (vs just slow loading) requires watching for changes. Duplicate detection helps confirm you've hit the end.

The trade-offs: URL navigation is most reliable but requires discovering the pattern. Clicking is most like human behavior but slower. Infinite scroll is the most complex to handle correctly.

### Validation Considerations

When scraping data, think about:
- **Completeness**: Are you getting the expected number of fields? Missing data might indicate extraction logic issues or page structure changes.
- **Consistency**: Do the data types match expectations? Unexpected formats suggest parsing problems.
- **Duplicates**: Especially with pagination, tracking what you've already seen prevents double-counting.
- **Sampling**: Spot-checking extracted data against the source confirms your extraction logic is working.

You're building a mental model of what "good data" looks like and checking against it.

## When Things Don't Work: Adaptation Strategies

### Elements Aren't Where You Expect

The accessibility tree shows what's in the DOM right now. If an element isn't there, it might:
- Not have loaded yet (dynamic content)
- Be hidden or in a collapsed section
- Use different markup than you expected
- Be called something else than you searched for

You can try adjusting depth, waiting a bit, expanding sections, or using different search terms with `find`. Sometimes taking a screenshot confirms whether it's truly missing vs just not in the part of the tree you're looking at.

### Interactions Fail

When clicking or filling a form doesn't work:
- The element might not be interactive yet (still loading, disabled state)
- Something could be covering it (modal, popup, overlay)
- The interaction method might not match what the site expects (some sites need clicks, some need form submission)
- JavaScript errors might be preventing normal behavior

`read_console_messages` and `read_network_requests` often reveal what's breaking. Console errors show JavaScript problems, network requests show failed API calls. A screenshot can show overlays or unexpected page state.

### Timing and Dynamic Content

Modern sites load content asynchronously. What you see in `read_page` is what's in the DOM right then - if content loads later, you need to account for that.

The `wait` action gives things time to settle. `wait_for` is smarter - it waits for specific text to appear (or disappear), so you're waiting just as long as needed rather than guessing at delays.

Reading the page again after waiting confirms whether content loaded. If it didn't, that tells you something about whether it's actually loading vs whether you're looking in the wrong place.

### Detection and Rate Limiting

Sites that care about bots look for patterns: repetitive timing, rapid requests, consistent behavior. If you're getting blocked or seeing CAPTCHAs:
- Varying timing between actions makes automation less obvious
- Respecting the site's robots.txt shows you're aware of their preferences
- Monitoring for block pages or CAPTCHA challenges lets you adapt

This is a judgment call - extracting data from your own company's site is different from aggressive scraping of third-party content.

## Debugging Perspective

When automation isn't working, you're debugging the gap between your mental model and reality:

**Structure mismatches**: What you think is on the page vs what `read_page` shows. Screenshots and the accessibility tree together reveal the truth.

**State assumptions**: Assuming content is loaded when it isn't, or that an action succeeded when it failed. Reading the page state after actions confirms what actually happened.

**Tool mismatches**: Choosing an approach that doesn't fit the problem. Playwright failing on ambiguous selectors suggests Chrome might handle it better. Chrome being too slow suggests Playwright for the repetitive parts.

**Timing gaps**: Acting before the page is ready. Console and network monitoring show what's still in progress.

The debugging process is usually: observe symptoms → check state/structure → adjust approach → verify. Sometimes that means switching tools, sometimes changing timing, sometimes rethinking the extraction logic.

## Multi-Page and Complex Workflows

When work spans multiple pages or steps:

**Mapping the territory**: Understanding the full workflow before automating helps spot decision points, data dependencies, and potential failure modes. You might discover that pages 2 and 3 could be skipped, or that certain data from page 1 is needed on page 3.

**State across pages**: Each page navigation might lose context unless you're explicitly tracking it. Data extracted on page 1 needs somewhere to live while you navigate to page 2. The workflow becomes: extract → store → navigate → extract → combine.

**Failure recovery**: If something breaks on page 3 of a 5-page workflow, you don't want to start over. Checkpointing progress (even just logging what's been completed) lets you resume rather than repeat.

**Tab management**: Claude in Chrome can work across multiple tabs (`tabs_context_mcp`, `tabs_create_mcp`). This lets you compare information across sources or run parallel workflows, though it adds complexity in tracking which tab is which.

## Observing and Adapting

The best automation comes from:
- Watching what actually happens rather than assuming
- Testing interactions before building complex workflows
- Being willing to switch approaches when something isn't working
- Understanding that websites change and automation needs to adapt

Console messages (`read_console_messages`) surface JavaScript errors and warnings - often the first sign something broke. Network requests (`read_network_requests`) show API calls and failures - crucial for sites that load data asynchronously.

Screenshots verify that what you think is happening actually is - they're your ground truth when the accessibility tree isn't telling the full story.

The tools work best when you're thinking about the problem characteristics and choosing approaches that fit, rather than following rigid procedures.

## Example Scenarios: Thinking Through Problems

### Authentication Flow
You need to log into a site. The typical pattern:
- Navigate to login page
- `read_page` reveals the form structure - where's the username field? Password? Submit button?
- You could use `form_input` with the username and password refs (fast, simple)
- Or click into each field and type (slower, but matches human behavior if the site is sensitive to automation)
- Submit is usually a button click with its ref
- After submission, `read_page` tells you if you're logged in (did the page change? Are there error messages?)
- 2FA complicates this - you might need to wait for a code, detect the 2FA page, handle that flow

The decision points: batch form filling vs sequential? How to verify success? What if 2FA appears?

### Search and Filter
You want to search for something on a site:
- `read_page` to find the search input and any filter controls
- Enter search terms (form_input or type action)
- Some sites auto-search as you type, some need a button click, some need Enter
- Wait for results to load - `wait_for` text appearing, or just wait and re-read the page
- Extract the results using whatever method fits the structure
- Handle "no results" gracefully (it's not an error, just empty data)

The variability: when does search trigger? How do you know results loaded? What if there are filters to apply first?

### Collecting Paginated Data
You need all the products from a category, but they're split across pages:
- Start at page 1, `read_page` to understand the data structure
- Extract current page's data
- Find pagination (numbered links? next button? URL pattern?)
- Navigate to next page (click or URL navigation depending on pattern)
- Extract that page's data
- Continue until you hit the last page (next button gone? URL pattern breaks?)
- Deduplicate if needed (sometimes pages overlap)

The judgments: is URL pattern more reliable than clicking? How do you detect the end? How do you validate you got everything?

## Core Principles Worth Remembering

**Structure reveals possibility**: The accessibility tree shows you what's actually there and gives you stable references to work with. Start there.

**Tools have personalities**: Chrome handles ambiguity and exploration well. Playwright handles speed and repetition well. Neither is always right.

**Verification matters**: `read_page` after actions tells you what actually happened vs what you hoped would happen.

**Adaptation over procedure**: When something doesn't work, understanding why (timing? structure? tool mismatch?) points toward what to try next.

**Screenshots confirm, refs interact**: Visual confirmation is valuable, but you interact through references and structure.

You're building a model of how the page works, testing that model with interactions, and refining based on what you observe. The tools are there to help you do that efficiently.