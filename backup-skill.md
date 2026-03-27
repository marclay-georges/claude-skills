---
name: browser-automation
description: Expert browser control and web scraping using accessibility trees, element references, and intelligent tool selection. Use when the user needs to interact with websites, extract data, automate web workflows, fill forms, or scrape content. Combines Claude in Chrome's flexibility with Playwright's speed based on task characteristics.
---

# Browser Automation Expert

This skill helps you understand browser automation tools and techniques, focusing on accessibility trees and element references as your primary navigation method.

## Architecture: Shared Browser via CDP

Claude in Chrome and Playwright control the **same browser instance**. Chrome Debugger is launched via `open '/Applications/Chrome Debugger.app'` with `--remote-debugging-port=9222`, and Playwright attaches to it over CDP. This means:
- Both tools see the same tabs, cookies, localStorage, and session state
- Actions taken by one tool are immediately visible to the other
- There is no separate "headless" Playwright browser — everything happens in the visible Chrome window
- You can watch Playwright's actions in real time

**Critical: Refs are not interchangeable.** Playwright uses `e8`, `e45` style refs. Chrome uses `ref_1`, `ref_2` style refs. You cannot pass a Playwright ref to a Chrome tool or vice versa. When switching tools mid-workflow, you must re-read the page with the new tool to get its own refs.

### Tool Selection: Playwright First, Chrome as Fallback

**Playwright is the default.** It's more efficient for interactions (single tool calls vs multiple), returns incremental snapshots after actions, and handles most workflows well. Use `browser_snapshot` for reading state, `browser_click`/`browser_type`/`browser_fill_form` for interactions.

**Chrome (Claude in Chrome) is the fallback** for when Playwright struggles — ambiguous selectors causing strict mode failures, complex discovery tasks where `find` tool's natural language matching helps, or when you need `read_page`'s filtering options (`filter: "interactive"`, `ref_id` scoping, depth control) to manage context on complex pages.

The handoff pattern when switching tools: Tool A acts → Tool B reads page (gets its own refs) → Tool B acts.

## Launching the Browser (Playwright)

Before Playwright can connect, Chrome must be running with remote debugging enabled. The dedicated launcher for this is:

```
/Applications/Chrome Debugger.app
```

**To open it**, use the `osascript` tool:

```applescript
tell application "/Applications/Chrome Debugger.app" to activate
```

Or via bash:

```bash
open -a "/Applications/Chrome Debugger.app"
```

This launches Chrome with `--remote-debugging-port=9222`, which is the port Playwright connects to. If Playwright fails to connect (connection refused, no browser found), the most likely cause is that Chrome Debugger.app isn't running. Open it first, then proceed with Playwright tool calls.

**Claude in Chrome** does not require this step — it has its own browser connection managed by the extension.

### If Playwright connects but hangs (CDP session corruption)

A subtler failure mode: Playwright reports `ws connected` but then times out with no further progress. This looks like a version mismatch but isn't — the WebSocket handshake succeeds, then Chrome silently drops the CDP protocol exchange. The cause is a stale singleton session in the Chrome Debug Profile. Chrome restores the same session UUID across relaunches, and if that session's internal state is corrupted, every connection attempt hangs identically regardless of Playwright version.

**Diagnosis**: Check whether the session UUID is the same across Chrome restarts:
```bash
curl -s http://localhost:9222/json/version | python3 -c "import json,sys; print(json.load(sys.stdin)['webSocketDebuggerUrl'])"
```
If you kill and relaunch Chrome Debugger.app and get back the same UUID, the profile singleton is stale.

**Fix**: Kill Chrome Debugger, wipe the singleton files, relaunch:
```bash
kill $(ps aux | grep 'remote-debugging-port=9222' | grep -v grep | awk '{print $2}')
rm -f ~/Library/Application\ Support/Google/Chrome\ Debug\ Profile/SingletonCookie
rm -f ~/Library/Application\ Support/Google/Chrome\ Debug\ Profile/SingletonLock
rm -f ~/Library/Application\ Support/Google/Chrome\ Debug\ Profile/SingletonSocket
```
Then relaunch via osascript: `tell application "/Applications/Chrome Debugger.app" to activate`. A successful fix produces a new UUID on the next `curl` check, and Playwright connects cleanly.

## Core Insight: Structure Before Vision

The accessibility tree reveals page structure in a way that's both machine-readable and human-understandable. When you use `browser_snapshot` or `read_page` first, you get:
- Element references that are stable and reliable for interaction
- Understanding of the page's semantic structure
- Text content and interactive elements without visual rendering
- A foundation for making informed decisions about what to do next

Screenshots are valuable for verification and spatial understanding, but the accessibility tree gives you actionable references. Think of it as reading the blueprint before decorating the room.

## Understanding Page Structure

When approaching a new page, `browser_snapshot` (Playwright) or `read_page` (Chrome) gives you the lay of the land. You're seeing:
- What elements exist and how they're organized
- Which elements are interactive (buttons, links, form fields)
- Text content and labels that indicate purpose
- Reference IDs that let you interact with specific elements

The depth parameter on `read_page` controls how deep into the tree you go. Deeper gives more detail but more output. If you're getting truncated output or need to focus, you can:
- Lower the depth to see less (8-10 for overview)
- Use a ref_id to focus on a specific part of the page
- Filter to "interactive" to see only actionable elements (but see caveats above about label stripping)

### Page Complexity and Snapshot Size

Not all pages are created equal when it comes to accessibility tree size. A login form might have a few dozen elements. The Amazon homepage has over 4,000 elements, 540+ interactive elements, and a DOM depth of 25. When you run `browser_snapshot` or `read_page` against a page like that, the result can exceed context limits entirely — you get a "Tool result too large for context" message and the output is saved to disk instead, meaning you spent a tool call and got nothing in your working context.

The variability is dramatic. Here's what we've observed on real pages:

| Page | Elements | Interactive | Depth |
|------|----------|-------------|-------|
| example.com | 13 | 1 | 3 |
| Amazon homepage | ~4,000 | ~540 | 25 |

A quick JavaScript probe can tell you what kind of page you're dealing with before you commit to a full read:

```javascript
(() => {
  const all = document.querySelectorAll('*');
  const interactive = document.querySelectorAll(
    'a, button, input, select, textarea, [role="button"], [role="link"], [tabindex]'
  );
  const depth = (el, d = 0) => {
    let max = d;
    for (const c of el.children) max = Math.max(max, depth(c, d + 1));
    return max;
  };
  return JSON.stringify({
    totalElements: all.length,
    interactiveElements: interactive.length,
    maxDOMDepth: depth(document.body),
    bodyChildCount: document.body.children.length,
    textLength: document.body.innerText.length
  });
})()
```

When the numbers suggest a heavy page, you have options: lower the depth on `read_page`, focus on a specific subtree via `ref_id`, save the snapshot to a file with the `filename` parameter and selectively read portions, or skip the tree entirely and go straight to targeted JavaScript extraction.

When the page is clearly simple — or when you already know the site from prior experience — the probe is unnecessary overhead. It's a tool for unfamiliar territory.

## The Automation Loop: Act → Read State → Act

Every interaction should follow this rhythm: **read state → act → read state again**. After every meaningful action (click, form fill, navigation), read the page state to confirm what actually happened before deciding what to do next.

This isn't optional verification — it's how you maintain an accurate model of the page. Pages change asynchronously, actions can fail silently, and elements can appear or disappear. The agent that reads state after every action catches problems immediately. The one that chains blind actions discovers failures three steps too late.

### Context-Efficient State Reading

Full accessibility tree dumps are expensive — they can consume significant context on complex pages. Use a tiered approach, choosing the lightest tool that gives you what you need:

**`browser_snapshot`** (Playwright) — the default for both exploration and verification. Returns a YAML representation of the page's accessibility structure with Playwright-style refs. Two key behaviors:
- **Initial exploration**: On content-rich pages (search results, dashboards, data tables, news feeds), a single snapshot can return enough structured data to answer questions or plan next steps without further navigation. On Hacker News, one snapshot returns all 30 stories with titles, points, authors, timestamps, and URLs. The upfront context cost pays for itself by avoiding multiple navigations.
- **Post-action verification**: After an interaction, `browser_snapshot` returns an **incremental** snapshot — only what changed. This is dramatically cheaper than a full snapshot. On a todo app, a post-action snapshot was ~8 lines vs ~50+ for a full read.
- **Caveat on complex pages**: On app-like pages with heavy UI chrome (GitHub, Instagram, web apps), the full snapshot is enormous — mostly navigation scaffolding, footer links, and decorative elements with actual content buried in noise. On these pages, consider targeted approaches instead.

**`read_page` with `filter: "interactive"` and low depth (3-5)** (Chrome) — returns only actionable elements (buttons, links, inputs). Dramatically smaller than a full read. **Known limitation**: on some sites (notably Instagram), the `interactive` filter strips element labels because accessible names live on child elements (images) rather than the parent links. You get `link [ref_1]` instead of `link "Instagram" [ref_1]`. Also missed checkboxes on some sites in testing. Useful for compact views but verify it captures what you need on each site.

**`read_page` with scoped `ref_id`** — when you know which section of the page you're working in. Pass a specific element's ref to get only its subtree. If you're filling a form, read just the form container, not the entire page.

**Full `read_page`** at default depth — reserve for when Chrome-side refs are needed with full labels, debugging unexpected behavior, or when `browser_snapshot` returned too much noise on an app-like page and you need Chrome's filtering options.

**`javascript_tool` with a targeted query** — the most token-efficient check of all. Use for binary verification: `document.querySelector('.error-message') !== null`, `document.title`, `window.location.href`. When you just need to confirm one specific thing happened, this beats reading any tree.

The principle: match your state-reading cost to the information you actually need. Use `browser_snapshot` aggressively for first contact with a page — you often get enough to work with. Then switch to incremental snapshots and targeted checks for the ongoing act → verify rhythm.

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

When you have a ref from `browser_snapshot`, `read_page`, or `find`, you can pass it directly to the appropriate tool. This is usually the smoothest path - you're referencing exactly what you identified in the structure.

If you don't have a ref (or it's not working), coordinates from a screenshot work but are more brittle - they break if window size changes or if the page layout shifts.

The general flow: identify element → get ref → interact using ref → verify the result. That last step matters — `browser_snapshot` after an interaction tells you if the page state changed as expected, without consuming the context that a full `read_page` would.

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
You already have the structure from `browser_snapshot` or `read_page` - you're just pulling out the pieces you need. Works well when content is in the DOM and has semantic meaning. You're pattern-matching against the tree structure to find repeating elements.

**get_page_text**
Gives you clean text content prioritized for articles and main content. Fast and simple, but you lose structural information. Good for reading, less good for structured data extraction. Think of it as "reader mode" - great for prose, limiting for tabular data.

**JavaScript Extraction**
The `javascript_tool` or `browser_evaluate` lets you query the DOM directly and return structured data as JSON. More powerful for complex extractions - you can traverse relationships, aggregate data, access page variables. You're writing the extraction logic yourself, which gives you precision but requires understanding the DOM structure.

Example: Extracting all products with their names and prices from cards - you'd query all product elements, map over them, extract specific fields, return as an array.

**Network Inspection**
Many modern sites load data through API calls rather than embedding it in HTML. `read_network_requests` shows you these calls. Sometimes you can extract the data from API responses directly, bypassing all the DOM manipulation. It's cleaner when it works, but requires the data to be in network requests rather than pre-rendered.

The trade-off is between simplicity (text extraction) and power (JavaScript/network inspection). Start simple, add complexity as needed.

### Programmatic Extraction and the Scratchpad

Browser automation sessions often involve accumulating information across multiple pages or steps — scraping product listings, collecting form data, gathering links to visit later. The natural instinct is to extract data and include it in the conversation as you go, but on data-heavy pages this can consume enormous amounts of context. A page with 50 product cards, each with a name, price, rating, and URL, might produce thousands of tokens of structured data. If you're mid-workflow with more pages to visit, that context is now occupied by data you've already collected rather than available for the work still ahead.

A scratchpad file offers a way to accumulate extracted data outside the context window. **Writes to the scratchpad should happen programmatically through scripts, not as directly generated output.** The difference matters: when you compose data as conversational output, every token of that data costs context. When a bash command writes to a file, the data goes to disk without passing through the context window.

The typical flow looks like this: JavaScript running in the page extracts structured data and returns it as a compact JSON result from `browser_evaluate` or `javascript_tool`. Then a bash command takes that data and appends it to a file on the filesystem — something like `/home/claude/scratchpad.md` or a `.jsonl` file if you prefer structured entries. The extraction happens in-page where the DOM is accessible, and the persistence happens through the shell where the filesystem is accessible.

Use the filesystem (`/home/claude/`) rather than browser storage APIs like `localStorage` or `sessionStorage`. Those are scoped per-origin — data written on amazon.com isn't readable when you navigate to github.com. For workflows that span multiple domains, which is most real browser automation, origin-scoped storage fragments your context across sites with no way to read it all back in one shot. A file on disk is domain-agnostic.

The scratchpad isn't meant to replace thinking. You still need some data in context to orient yourself — understanding page structure, deciding what to extract, verifying that extraction logic is working. The scratchpad is for the bulk output: the 50 product listings, the accumulated links, the data you've already processed and just need to store for later reference or final assembly.

When you need the data back, `view` or `cat` the scratchpad file brings it into context on demand — and you can be selective about it, reading just the last few entries or grepping for specific content rather than loading everything.

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

Modern sites load content asynchronously. What you see in `browser_snapshot` or `read_page` is what's in the DOM right then - if content loads later, you need to account for that.

The `wait` action gives things time to settle. `wait_for` is smarter - it waits for specific text to appear (or disappear), so you're waiting just as long as needed rather than guessing at delays.

A quick `browser_snapshot` after waiting confirms whether content loaded — if it didn't, that tells you something about whether it's actually loading vs whether you're looking in the wrong place.

### Detection and Rate Limiting

Sites that care about bots look for patterns: repetitive timing, rapid requests, consistent behavior. If you're getting blocked or seeing CAPTCHAs:
- Varying timing between actions makes automation less obvious
- Respecting the site's robots.txt shows you're aware of their preferences
- Monitoring for block pages or CAPTCHA challenges lets you adapt

This is a judgment call - extracting data from your own company's site is different from aggressive scraping of third-party content.

## Debugging Perspective

When automation isn't working, you're debugging the gap between your mental model and reality:

**Structure mismatches**: What you think is on the page vs what `browser_snapshot` shows. Screenshots and the accessibility tree together reveal the truth.

**State assumptions**: Assuming content is loaded when it isn't, or that an action succeeded when it failed. Reading the page state after actions confirms what actually happened.

**Tool mismatches**: Choosing an approach that doesn't fit the problem. Playwright failing on ambiguous selectors suggests Chrome might handle it better. Chrome being too slow suggests Playwright for the repetitive parts.

**Timing gaps**: Acting before the page is ready. Console and network monitoring show what's still in progress.

**Cross-tool filesystem gaps**: Playwright's `/tmp` and bash's `/tmp` are separate filesystem contexts. Saving a screenshot via Playwright and reading it via bash won't work. The working pattern is `page.screenshot({ type: 'png' })` returning a buffer, then `.toString('base64')` to pass image data through the tool return value rather than the filesystem.

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
- `browser_snapshot` reveals the form structure - where's the username field? Password? Submit button?
- You could use `browser_fill_form` or `form_input` with the refs (fast, simple)
- Or click into each field and type (slower, but matches human behavior if the site is sensitive to automation)
- Submit is usually a button click with its ref
- After submission, `browser_snapshot` tells you if you're logged in (did the page change? Are there error messages?)
- 2FA complicates this - you might need to wait for a code, detect the 2FA page, handle that flow

The decision points: batch form filling vs sequential? How to verify success? What if 2FA appears?

### Search and Filter
You want to search for something on a site:
- `browser_snapshot` to find the search input and any filter controls
- Enter search terms (`browser_type` or `form_input`)
- Some sites auto-search as you type, some need a button click, some need Enter
- Wait for results to load - `wait_for` text appearing, or just wait and re-read the page
- Extract the results using whatever method fits the structure
- Handle "no results" gracefully (it's not an error, just empty data)

The variability: when does search trigger? How do you know results loaded? What if there are filters to apply first?

### Collecting Paginated Data
You need all the products from a category, but they're split across pages:
- Start at page 1, `browser_snapshot` to understand the data structure
- Extract current page's data
- Find pagination (numbered links? next button? URL pattern?)
- Navigate to next page (click or URL navigation depending on pattern)
- Extract that page's data
- Continue until you hit the last page (next button gone? URL pattern breaks?)
- Deduplicate if needed (sometimes pages overlap)

The judgments: is URL pattern more reliable than clicking? How do you detect the end? How do you validate you got everything?

## Core Principles Worth Remembering

**Structure reveals possibility**: The accessibility tree shows you what's actually there and gives you stable references to work with. Start there.

**Playwright first, Chrome as fallback**: Playwright is more efficient for most workflows. Chrome shines when Playwright struggles with ambiguity or when you need its filtering options.

**Snapshot early, snapshot often**: `browser_snapshot` is your default state-reading tool. Use it for initial exploration and after every action. Escalate to `read_page` or targeted JS only when you need more.

**Refs don't cross tool boundaries**: Playwright refs and Chrome refs are incompatible. Re-read the page when switching tools.

**Verification matters**: `browser_snapshot` after every action that changes state. Not sometimes — every time. This is the difference between robust automation and fragile scripts.

**Context is finite**: Full tree dumps are expensive. Default to `browser_snapshot` for verification, filtered `read_page` for targeted interaction planning, and full `read_page` only for exploration or debugging. Match your state-reading cost to the information you actually need.

**Adaptation over procedure**: When something doesn't work, understanding why (timing? structure? tool mismatch?) points toward what to try next.

**Screenshots confirm, refs interact**: Visual confirmation is valuable, but you interact through references and structure.

You're building a model of how the page works, testing that model with interactions, and refining based on what you observe. The tools are there to help you do that efficiently.
