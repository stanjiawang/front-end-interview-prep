# Pragmatic UI Interview Study Guide

This guide is organized for the one-hour **Pragmatic Coding for UI** round. The first five exercises come from the recruiter’s recent question bank and should be practiced first.

> Default implementation style: semantic HTML, focused CSS, and framework-free JavaScript. Build the smallest working version first, then add accessibility, asynchronous states, performance, and cleanup.

## Table of Contents

1. [What the Round Evaluates](#round-evaluation)
2. [The 60-Minute Interview Strategy](#interview-strategy)
3. [Priority 1: Autocomplete](#autocomplete)
   - [Autocomplete requirements](#autocomplete-requirements)
   - [Autocomplete interview solution](#autocomplete-solution)
   - [Autocomplete follow-ups](#autocomplete-follow-ups)
4. [Priority 2: Accordion](#accordion)
   - [Native no-JavaScript solution](#accordion-native)
   - [Custom JavaScript solution](#accordion-custom)
   - [Accordion follow-ups](#accordion-follow-ups)
5. [Priority 3: Calendar](#calendar)
   - [Calendar interview solution](#calendar-solution)
   - [Calendar follow-ups](#calendar-follow-ups)
6. [Priority 4: LinkedIn Post](#linkedin-post)
   - [LinkedIn Post interview solution](#linkedin-post-solution)
   - [LinkedIn Post follow-ups](#linkedin-post-follow-ups)
7. [Priority 5: Calculator UI](#calculator-ui)
   - [Calculator interview solution](#calculator-solution)
   - [Unlimited undo](#calculator-undo)
   - [Calculator follow-ups](#calculator-follow-ups)
8. [Previously Reported LinkedIn Exercises](#reported-exercises)
   - [People You May Know](#people-you-may-know)
   - [Tooltip](#tooltip)
   - [Responsive Top Navigation](#top-navigation)
   - [Infinite Scroll](#infinite-scroll)
9. [Shared UI Foundations](#shared-foundations)
10. [Testing Checklist](#testing-checklist)
11. [Preparation Plan](#preparation-plan)
12. [English Interview Phrases](#english-phrases)

---

<a id="round-evaluation"></a>
## 1. What the Round Evaluates

The interviewer is evaluating more than the final screenshot:

1. Semantic HTML
2. CSS layout and responsive behavior
3. Clear UI state
4. DOM rendering and updates
5. Event handling and delegation
6. AJAX and race-condition handling
7. Loading, empty, error, and disabled states
8. Keyboard and screen-reader accessibility
9. Performance and cleanup
10. Communication, prioritization, and testing

### Pragmatic does not mean careless

A pragmatic solution:

- Delivers the core user flow early.
- Uses native browser behavior where possible.
- Keeps state, rendering, networking, and events understandable.
- Handles the highest-risk edge cases.
- Leaves advanced features as clearly explained follow-ups.

Avoid spending the first 30 minutes building abstractions or pixel-perfect decoration.

---

<a id="interview-strategy"></a>
## 2. The 60-Minute Interview Strategy

### Minutes 0–5: Clarify and scope

Ask only questions that change the implementation:

> Should I use plain HTML, CSS, and JavaScript?

> Is there an API or should I provide a mock?

> Which behaviors are required for the first working version?

> Should I support keyboard interaction and responsive layout?

> Should server updates be optimistic or pessimistic?

State your plan:

> I’ll first build the semantic structure and core interaction. Then I’ll add styling, asynchronous states, accessibility, and edge cases in that order.

### Minutes 5–12: Semantic HTML and state model

Write the minimum structure and name the state:

```js
const state = {
  items: [],
  loading: false,
  error: null,
};
```

### Minutes 12–30: Core behavior

Get the primary interaction working:

- Render initial data.
- Handle the main user action.
- Update state and DOM.
- Verify one normal path.

### Minutes 30–42: CSS and responsive behavior

Use:

- Flexbox for one-dimensional alignment.
- Grid for repeated cards or calendar cells.
- Natural wrapping before adding media queries.
- Visible `:focus-visible` styles.

### Minutes 42–52: Production states

Add the most relevant:

- Loading
- Empty
- Error and retry
- Disabled controls
- Duplicate-request prevention
- Stale-response protection

### Minutes 52–57: Accessibility and cleanup

Check:

- Native buttons and inputs
- Labels and accessible names
- Keyboard behavior
- Focus behavior
- `aria-live` for meaningful asynchronous status
- Listener, observer, timer, and request cleanup

### Minutes 57–60: Test and summarize

> I’ll test the normal flow, empty input, repeated actions, failure behavior, and keyboard interaction. With more time, I would add the following production improvements.

---

<a id="autocomplete"></a>
## 3. Priority 1: Autocomplete

Autocomplete is the highest-value exercise because it combines DOM work, AJAX, race conditions, performance, keyboard interaction, and accessibility.

<a id="autocomplete-requirements"></a>
### 3.1 Requirements

Build a reusable autocomplete component that:

- Uses a text input.
- Waits until the query reaches a minimum length.
- Limits the maximum query length.
- Debounces requests.
- Loads suggestions through AJAX.
- Handles loading, empty, and error states.
- Ignores or cancels stale requests.
- Supports mouse and keyboard selection.
- Uses accessible combobox semantics.
- Works without React or another framework.

### Clarifying questions

> What are the minimum and maximum query lengths?

> Is the API response ordered by relevance?

> Should selecting an option submit immediately or only fill the input?

> Should results be cached?

> Is free-form text allowed?

<a id="autocomplete-solution"></a>
### 3.2 Interview solution

#### HTML

```html
<div class="autocomplete" id="people-search">
  <label for="search-input">Search people</label>

  <input
    id="search-input"
    type="search"
    role="combobox"
    aria-autocomplete="list"
    aria-expanded="false"
    aria-controls="search-list"
    autocomplete="off"
    maxlength="50"
  />

  <ul id="search-list" role="listbox" hidden></ul>

  <p id="search-status" class="status" aria-live="polite"></p>
</div>
```

#### CSS

```css
* {
  box-sizing: border-box;
}

.autocomplete {
  position: relative;
  width: min(100%, 420px);
  font-family: system-ui, sans-serif;
}

.autocomplete label {
  display: block;
  margin-bottom: 6px;
  font-weight: 600;
}

.autocomplete input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #666;
  border-radius: 6px;
  font: inherit;
}

.autocomplete input:focus-visible {
  outline: 3px solid #0a66c2;
  outline-offset: 2px;
}

[role="listbox"] {
  position: absolute;
  z-index: 10;
  width: 100%;
  max-height: 260px;
  margin: 4px 0 0;
  padding: 4px 0;
  overflow-y: auto;
  list-style: none;
  background: white;
  border: 1px solid #aaa;
  border-radius: 6px;
  box-shadow: 0 6px 18px rgb(0 0 0 / 18%);
}

[role="option"] {
  padding: 10px 12px;
  cursor: pointer;
}

[role="option"][aria-selected="true"] {
  color: white;
  background: #0a66c2;
}

.status {
  min-height: 1.5em;
  margin: 6px 0 0;
}
```

#### JavaScript

```js
function debounce(fn, delay) {
  let timer = null; // Tracks the pending invocation.

  function debounced(...args) {
    clearTimeout(timer); // Restart the quiet period.

    timer = setTimeout(() => {
      timer = null;
      fn.apply(this, args); // Preserve context and arguments.
    }, delay);
  }

  debounced.cancel = () => {
    clearTimeout(timer);
    timer = null;
  };

  return debounced;
}

function createAutocomplete({
  root,
  search,
  onSelect,
  minLength = 2,
  maxLength = 50,
}) {
  const input = root.querySelector('[role="combobox"]');
  const list = root.querySelector('[role="listbox"]');
  const status = root.querySelector('[aria-live]');

  let options = [];      // Current suggestions.
  let activeIndex = -1;  // Keyboard-highlighted option.
  let requestId = 0;     // Identifies the newest request.
  let controller = null; // Cancels the previous request.

  function close() {
    options = [];
    activeIndex = -1;
    list.replaceChildren();
    list.hidden = true;
    input.setAttribute("aria-expanded", "false");
    input.removeAttribute("aria-activedescendant");
  }

  function setActive(index) {
    const elements = [...list.querySelectorAll('[role="option"]')];

    if (elements.length === 0) {
      activeIndex = -1;
      return;
    }

    activeIndex = (index + elements.length) % elements.length;

    elements.forEach((element, currentIndex) => {
      element.setAttribute(
        "aria-selected",
        String(currentIndex === activeIndex),
      );
    });

    const active = elements[activeIndex];
    input.setAttribute("aria-activedescendant", active.id);
    active.scrollIntoView({ block: "nearest" });
  }

  function render() {
    const fragment = document.createDocumentFragment();

    options.forEach((option, index) => {
      const item = document.createElement("li");

      item.id = `search-option-${index}`;
      item.dataset.index = String(index);
      item.setAttribute("role", "option");
      item.setAttribute("aria-selected", "false");
      item.textContent = option.label; // Avoid unsafe innerHTML.
      fragment.append(item);
    });

    list.replaceChildren(fragment);
    list.hidden = options.length === 0;
    input.setAttribute("aria-expanded", String(options.length > 0));

    status.textContent =
      options.length === 0
        ? "No suggestions found."
        : `${options.length} suggestions available.`;
  }

  function select(index) {
    const option = options[index];

    if (!option) {
      return;
    }

    input.value = option.label;
    close();
    status.textContent = `${option.label} selected.`;
    onSelect(option);
  }

  async function load() {
    const query = input.value.trim();

    if (query.length < minLength) {
      controller?.abort();
      close();
      status.textContent =
        query.length === 0
          ? ""
          : `Enter at least ${minLength} characters.`;
      return;
    }

    if (query.length > maxLength) {
      close();
      status.textContent = `Query cannot exceed ${maxLength} characters.`;
      return;
    }

    controller?.abort(); // Cancel the older network request.
    controller = new AbortController();
    const currentRequest = ++requestId;

    status.textContent = "Loading suggestions…";

    try {
      const result = await search(query, controller.signal);

      // Ignore a response that is no longer current.
      if (currentRequest !== requestId) {
        return;
      }

      options = result;
      activeIndex = -1;
      render();
    } catch (error) {
      if (error.name === "AbortError") {
        return;
      }

      close();
      status.textContent = "Unable to load suggestions.";
    }
  }

  const debouncedLoad = debounce(load, 250);

  function handleKeyDown(event) {
    if (event.key === "ArrowDown" && options.length > 0) {
      event.preventDefault();
      setActive(activeIndex + 1);
    } else if (event.key === "ArrowUp" && options.length > 0) {
      event.preventDefault();
      setActive(activeIndex - 1);
    } else if (event.key === "Enter" && activeIndex >= 0) {
      event.preventDefault();
      select(activeIndex);
    } else if (event.key === "Escape") {
      close();
    }
  }

  function handlePointerDown(event) {
    const option = event.target.closest('[role="option"][data-index]');

    if (!option || !list.contains(option)) {
      return;
    }

    // Prevent the input from losing focus before selection.
    event.preventDefault();
    select(Number(option.dataset.index));
  }

  input.addEventListener("input", debouncedLoad);
  input.addEventListener("keydown", handleKeyDown);
  list.addEventListener("pointerdown", handlePointerDown);

  return function destroy() {
    debouncedLoad.cancel();
    controller?.abort();
    input.removeEventListener("input", debouncedLoad);
    input.removeEventListener("keydown", handleKeyDown);
    list.removeEventListener("pointerdown", handlePointerDown);
  };
}
```

#### Example API

```js
async function searchPeople(query, signal) {
  const url = new URL("/api/people", window.location.origin);
  url.searchParams.set("q", query);
  url.searchParams.set("limit", "8");

  const response = await fetch(url, { signal });

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  const data = await response.json();
  return data.items;
}

const destroyAutocomplete = createAutocomplete({
  root: document.querySelector("#people-search"),
  search: searchPeople,
  onSelect(option) {
    console.log("Selected:", option.id);
  },
});
```

### Interview explanation

> I keep DOM focus in the input and expose the highlighted option through `aria-activedescendant`. The request is debounced, and I both abort the previous request and compare request IDs so an older response cannot replace newer results. Rendering uses `textContent` because suggestion labels may come from the server.

<a id="autocomplete-follow-ups"></a>
### 3.3 Follow-ups

- Client cache by normalized query
- Request deduplication
- Result highlighting without unsafe HTML
- Server pagination
- Recent searches
- Multiple categories
- Virtualization for many options
- Composition events for input methods
- Outside-click dismissal
- Loading indicator that does not over-announce
- Retry button

---

<a id="accordion"></a>
## 4. Priority 2: Accordion

This exercise tests semantic markup, native browser behavior, DOM relationships, delegation, and keyboard interaction.

<a id="accordion-native"></a>
### 4.1 Native no-JavaScript solution

Use `<details>` and `<summary>` when the requirements match native disclosure behavior:

```html
<section class="faq" aria-labelledby="faq-title">
  <h2 id="faq-title">Frequently asked questions</h2>

  <details>
    <summary>Who can see my profile?</summary>
    <p>Your profile visibility depends on your privacy settings.</p>
  </details>

  <details>
    <summary>How do I manage notifications?</summary>
    <p>You can update notification preferences in Settings.</p>
  </details>
</section>
```

```css
.faq {
  width: min(100%, 680px);
  font-family: system-ui, sans-serif;
}

.faq details {
  border-bottom: 1px solid #ccc;
}

.faq summary {
  padding: 16px 4px;
  font-weight: 600;
  cursor: pointer;
}

.faq summary:focus-visible {
  outline: 3px solid #0a66c2;
  outline-offset: 2px;
}

.faq details > :not(summary) {
  margin: 0;
  padding: 0 4px 16px;
}
```

**Spoken answer**

> If the required behavior is a standard disclosure, I would prefer `details` and `summary`. They provide native semantics and keyboard behavior without JavaScript. I would only build a custom accordion if the product requires behavior the native elements cannot provide consistently.

<a id="accordion-custom"></a>
### 4.2 Custom JavaScript solution

#### HTML

```html
<section id="accordion" class="accordion" aria-labelledby="accordion-title">
  <h2 id="accordion-title">Product help</h2>

  <div class="accordion-item">
    <h3>
      <button
        type="button"
        data-accordion-trigger
        aria-expanded="true"
        aria-controls="panel-profile"
        id="trigger-profile"
      >
        Profile
      </button>
    </h3>

    <div
      id="panel-profile"
      role="region"
      aria-labelledby="trigger-profile"
    >
      <p>Manage your photo, headline, and experience.</p>
      <img src="profile-example.png" alt="Profile editor example" />
    </div>
  </div>

  <div class="accordion-item">
    <h3>
      <button
        type="button"
        data-accordion-trigger
        aria-expanded="false"
        aria-controls="panel-privacy"
        id="trigger-privacy"
      >
        Privacy
      </button>
    </h3>

    <div
      id="panel-privacy"
      role="region"
      aria-labelledby="trigger-privacy"
      hidden
    >
      <p>Choose who can see your activity.</p>
    </div>
  </div>
</section>
```

#### CSS

```css
.accordion {
  width: min(100%, 680px);
  font-family: system-ui, sans-serif;
}

.accordion h3 {
  margin: 0;
}

[data-accordion-trigger] {
  width: 100%;
  padding: 16px;
  color: inherit;
  text-align: left;
  background: white;
  border: 0;
  border-bottom: 1px solid #ccc;
  cursor: pointer;
  font: inherit;
  font-weight: 600;
}

[data-accordion-trigger]::after {
  content: "+";
  float: right;
}

[data-accordion-trigger][aria-expanded="true"]::after {
  content: "−";
}

[data-accordion-trigger]:focus-visible {
  outline: 3px solid #0a66c2;
  outline-offset: -3px;
}

[role="region"] {
  padding: 16px;
}

[role="region"] img {
  max-width: 100%;
  height: auto;
}
```

#### JavaScript

```js
function createAccordion(root, { singleOpen = true } = {}) {
  function getButtons() {
    return [...root.querySelectorAll("[data-accordion-trigger]")];
  }

  function setExpanded(button, expanded) {
    const panelId = button.getAttribute("aria-controls");
    const panel = document.getElementById(panelId);

    button.setAttribute("aria-expanded", String(expanded));
    panel.hidden = !expanded;
  }

  function toggle(button) {
    const willOpen =
      button.getAttribute("aria-expanded") !== "true";

    if (singleOpen && willOpen) {
      for (const other of getButtons()) {
        if (other !== button) {
          setExpanded(other, false);
        }
      }
    }

    setExpanded(button, willOpen);
  }

  function handleClick(event) {
    const button = event.target.closest("[data-accordion-trigger]");

    if (!button || !root.contains(button)) {
      return;
    }

    toggle(button);
  }

  function handleKeyDown(event) {
    const button = event.target.closest("[data-accordion-trigger]");

    if (!button || !root.contains(button)) {
      return;
    }

    const buttons = getButtons();
    const index = buttons.indexOf(button);
    let nextIndex = null;

    if (event.key === "ArrowDown") {
      nextIndex = (index + 1) % buttons.length;
    } else if (event.key === "ArrowUp") {
      nextIndex = (index - 1 + buttons.length) % buttons.length;
    } else if (event.key === "Home") {
      nextIndex = 0;
    } else if (event.key === "End") {
      nextIndex = buttons.length - 1;
    }

    if (nextIndex !== null) {
      event.preventDefault();
      buttons[nextIndex].focus();
    }
  }

  root.addEventListener("click", handleClick);
  root.addEventListener("keydown", handleKeyDown);

  return function destroy() {
    root.removeEventListener("click", handleClick);
    root.removeEventListener("keydown", handleKeyDown);
  };
}

createAccordion(document.querySelector("#accordion"));
```

### Interview explanation

> I use buttons inside headings because the headers perform actions rather than navigation. Each button exposes its state through `aria-expanded` and identifies its panel with `aria-controls`. Events are delegated from the stable accordion container, so dynamically added items also work.

<a id="accordion-follow-ups"></a>
### 4.3 Follow-ups

- Multiple panels open
- Exactly one panel open
- Nested accordions
- Animated height
- Deep-link to a panel
- Dynamic content
- Preserve state in the URL
- Use links only when the header navigates
- Native `<details>` compatibility and styling trade-offs

---

<a id="calendar"></a>
## 5. Priority 3: Calendar

This exercise tests date logic, grid layout, rendering, state, and navigation.

### Clarifying questions

> Should the week begin on Sunday or Monday?

> Do I need date selection or only month navigation?

> Should adjacent-month dates be visible?

> Which locale and time zone should I use?

<a id="calendar-solution"></a>
### 5.1 Interview solution

#### HTML

```html
<section id="calendar" class="calendar" aria-labelledby="month-label">
  <header class="calendar-header">
    <button type="button" data-action="previous" aria-label="Previous month">
      ‹
    </button>

    <h2 id="month-label"></h2>

    <button type="button" data-action="next" aria-label="Next month">
      ›
    </button>
  </header>

  <div class="weekdays" aria-hidden="true">
    <span>Sun</span>
    <span>Mon</span>
    <span>Tue</span>
    <span>Wed</span>
    <span>Thu</span>
    <span>Fri</span>
    <span>Sat</span>
  </div>

  <div id="calendar-grid" class="calendar-grid"></div>

  <p id="calendar-status" class="sr-only" aria-live="polite"></p>
</section>
```

#### CSS

```css
* {
  box-sizing: border-box;
}

.calendar {
  width: min(100%, 420px);
  padding: 16px;
  font-family: system-ui, sans-serif;
  border: 1px solid #ccc;
  border-radius: 12px;
}

.calendar-header {
  display: grid;
  grid-template-columns: 44px 1fr 44px;
  align-items: center;
}

.calendar-header h2 {
  margin: 0;
  text-align: center;
  font-size: 1.2rem;
}

.calendar-header button,
.date-button {
  min-width: 40px;
  min-height: 40px;
  font: inherit;
}

.weekdays,
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 4px;
}

.weekdays {
  margin: 16px 0 8px;
  text-align: center;
  font-size: 0.8rem;
  font-weight: 600;
}

.calendar-cell {
  aspect-ratio: 1;
}

.date-button {
  width: 100%;
  height: 100%;
  background: white;
  border: 0;
  border-radius: 50%;
  cursor: pointer;
}

.date-button:hover {
  background: #eef3f8;
}

.date-button[aria-current="date"] {
  color: white;
  background: #0a66c2;
}

.date-button[aria-pressed="true"] {
  outline: 3px solid #004182;
}

.date-button:focus-visible,
.calendar-header button:focus-visible {
  outline: 3px solid #0a66c2;
  outline-offset: 2px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
}
```

#### JavaScript

```js
function createCalendar(root, initialDate = new Date()) {
  const label = root.querySelector("#month-label");
  const grid = root.querySelector("#calendar-grid");
  const status = root.querySelector("#calendar-status");

  let year = initialDate.getFullYear();
  let month = initialDate.getMonth();
  let selected = null;

  const monthFormatter = new Intl.DateTimeFormat(undefined, {
    month: "long",
    year: "numeric",
  });

  const dateFormatter = new Intl.DateTimeFormat(undefined, {
    weekday: "long",
    month: "long",
    day: "numeric",
    year: "numeric",
  });

  function sameDate(left, right) {
    return (
      left?.getFullYear() === right?.getFullYear() &&
      left?.getMonth() === right?.getMonth() &&
      left?.getDate() === right?.getDate()
    );
  }

  function render() {
    const firstWeekday = new Date(year, month, 1).getDay();
    const daysInMonth = new Date(year, month + 1, 0).getDate();
    const today = new Date();
    const fragment = document.createDocumentFragment();

    label.textContent = monthFormatter.format(new Date(year, month, 1));

    // Add empty cells before the first day.
    for (let i = 0; i < firstWeekday; i += 1) {
      const empty = document.createElement("div");
      empty.className = "calendar-cell";
      empty.setAttribute("aria-hidden", "true");
      fragment.append(empty);
    }

    for (let day = 1; day <= daysInMonth; day += 1) {
      const date = new Date(year, month, day);
      const cell = document.createElement("div");
      const button = document.createElement("button");

      cell.className = "calendar-cell";
      button.type = "button";
      button.className = "date-button";
      button.dataset.day = String(day);
      button.textContent = String(day);
      button.setAttribute("aria-label", dateFormatter.format(date));
      button.setAttribute("aria-pressed", String(sameDate(date, selected)));

      if (sameDate(date, today)) {
        button.setAttribute("aria-current", "date");
      }

      cell.append(button);
      fragment.append(cell);
    }

    grid.replaceChildren(fragment);
  }

  function changeMonth(offset) {
    const date = new Date(year, month + offset, 1);
    year = date.getFullYear();
    month = date.getMonth();
    render();
    status.textContent = label.textContent;
  }

  function handleClick(event) {
    const action = event.target.closest("[data-action]");

    if (action && root.contains(action)) {
      changeMonth(action.dataset.action === "next" ? 1 : -1);
      return;
    }

    const dateButton = event.target.closest("[data-day]");

    if (!dateButton || !grid.contains(dateButton)) {
      return;
    }

    selected = new Date(year, month, Number(dateButton.dataset.day));
    render();
    status.textContent = `${dateFormatter.format(selected)} selected.`;
  }

  root.addEventListener("click", handleClick);
  render();

  return function destroy() {
    root.removeEventListener("click", handleClick);
  };
}

createCalendar(document.querySelector("#calendar"));
```

### Interview explanation

> The calendar state only contains the displayed year and month plus the selected date. I calculate the first weekday and number of days, then render a seven-column grid. `Date(year, month + 1, 0)` gives the final day of the displayed month and naturally handles leap years.

<a id="calendar-follow-ups"></a>
### 5.2 Follow-ups

- Monday-first locale
- Adjacent-month dates
- Arrow-key grid navigation
- Roving `tabindex`
- Date range
- Disabled dates
- Events
- Time zones and DST
- Month/year picker
- Server availability API
- Unit tests for leap years and year boundaries

---

<a id="linkedin-post"></a>
## 6. Priority 4: LinkedIn Post

This exercise tests semantic content, visual states, optimistic updates, form behavior, dynamic comments, and safe rendering.

<a id="linkedin-post-solution"></a>
### 6.1 Interview solution

#### HTML

```html
<article id="post" class="post" data-post-id="post-123">
  <header class="post-author">
    <img src="avatar.jpg" alt="" width="48" height="48" />
    <div>
      <h2>Alex Chen</h2>
      <p>Staff Software Engineer</p>
    </div>
  </header>

  <p class="post-body">
    We are hiring engineers to build accessible professional experiences.
  </p>

  <p class="post-counts">
    <span id="like-count">12 likes</span>
    <span id="comment-count">0 comments</span>
  </p>

  <div class="post-actions">
    <button
      id="like-button"
      type="button"
      aria-pressed="false"
      data-action="like"
    >
      Like
    </button>
  </div>

  <section aria-labelledby="comments-title">
    <h3 id="comments-title">Comments</h3>

    <form id="comment-form">
      <label for="comment-input">Add a comment</label>
      <textarea id="comment-input" maxlength="500" required></textarea>
      <button type="submit">Post comment</button>
    </form>

    <ul id="comment-list" class="comment-list"></ul>
  </section>

  <p id="post-status" aria-live="polite"></p>
</article>
```

#### CSS

```css
* {
  box-sizing: border-box;
}

.post {
  width: min(100%, 600px);
  padding: 16px;
  font-family: system-ui, sans-serif;
  background: white;
  border: 1px solid #ddd;
  border-radius: 10px;
}

.post-author {
  display: flex;
  gap: 12px;
  align-items: center;
}

.post-author img {
  border-radius: 50%;
  object-fit: cover;
}

.post-author h2,
.post-author p {
  margin: 0;
}

.post-body {
  line-height: 1.5;
}

.post-counts,
.post-actions {
  display: flex;
  gap: 16px;
  padding: 10px 0;
  border-bottom: 1px solid #ddd;
}

#like-button {
  padding: 8px 14px;
  color: #444;
  background: transparent;
  border: 0;
  border-radius: 4px;
  font: inherit;
  font-weight: 600;
  cursor: pointer;
}

#like-button[aria-pressed="true"] {
  color: #0a66c2;
  background: #eef3f8;
}

#like-button:focus-visible,
#comment-form button:focus-visible,
#comment-input:focus-visible {
  outline: 3px solid #0a66c2;
  outline-offset: 2px;
}

#comment-form {
  display: grid;
  gap: 8px;
}

#comment-input {
  min-height: 80px;
  resize: vertical;
  font: inherit;
}

.comment-list {
  padding: 0;
  list-style: none;
}

.comment-list li {
  margin-top: 12px;
  padding: 10px;
  background: #f3f2ef;
  border-radius: 8px;
}
```

#### JavaScript

```js
function createPost(root, api) {
  const likeButton = root.querySelector("#like-button");
  const likeCount = root.querySelector("#like-count");
  const commentCount = root.querySelector("#comment-count");
  const form = root.querySelector("#comment-form");
  const input = root.querySelector("#comment-input");
  const list = root.querySelector("#comment-list");
  const status = root.querySelector("#post-status");

  const state = {
    liked: false,
    likes: 12,
    comments: [],
    liking: false,
    commenting: false,
  };

  function renderCounts() {
    likeButton.setAttribute("aria-pressed", String(state.liked));
    likeButton.textContent = state.liked ? "Liked" : "Like";
    likeCount.textContent = `${state.likes} ${state.likes === 1 ? "like" : "likes"}`;
    commentCount.textContent =
      `${state.comments.length} ` +
      `${state.comments.length === 1 ? "comment" : "comments"}`;
  }

  function renderComments() {
    const fragment = document.createDocumentFragment();

    for (const comment of state.comments) {
      const item = document.createElement("li");
      const author = document.createElement("strong");
      const body = document.createElement("p");

      author.textContent = comment.author;
      body.textContent = comment.body; // Treat user text as text, not HTML.
      item.append(author, body);
      fragment.append(item);
    }

    list.replaceChildren(fragment);
  }

  async function toggleLike() {
    if (state.liking) {
      return;
    }

    state.liking = true;
    likeButton.disabled = true;

    // Save state so an optimistic update can be rolled back.
    const previousLiked = state.liked;
    const previousLikes = state.likes;

    state.liked = !state.liked;
    state.likes += state.liked ? 1 : -1;
    renderCounts();

    try {
      await api.setLiked(root.dataset.postId, state.liked);
      status.textContent = state.liked ? "Post liked." : "Like removed.";
    } catch {
      state.liked = previousLiked;
      state.likes = previousLikes;
      renderCounts();
      status.textContent = "Unable to update the Like. Please try again.";
    } finally {
      state.liking = false;
      likeButton.disabled = false;
    }
  }

  async function addComment(event) {
    event.preventDefault();

    const body = input.value.trim();

    if (body === "" || state.commenting) {
      return;
    }

    state.commenting = true;
    input.disabled = true;
    form.querySelector('button[type="submit"]').disabled = true;
    status.textContent = "Posting comment…";

    try {
      const comment = await api.addComment(root.dataset.postId, body);
      state.comments.push(comment);
      input.value = "";
      renderComments();
      renderCounts();
      status.textContent = "Comment posted.";
    } catch {
      status.textContent = "Unable to post the comment.";
    } finally {
      state.commenting = false;
      input.disabled = false;
      form.querySelector('button[type="submit"]').disabled = false;
      input.focus();
    }
  }

  likeButton.addEventListener("click", toggleLike);
  form.addEventListener("submit", addComment);
  renderCounts();

  return function destroy() {
    likeButton.removeEventListener("click", toggleLike);
    form.removeEventListener("submit", addComment);
  };
}
```

#### Example API boundary

```js
const postApi = {
  async setLiked(postId, liked) {
    const response = await fetch(`/api/posts/${postId}/like`, {
      method: liked ? "PUT" : "DELETE",
    });

    if (!response.ok) {
      throw new Error(`Request failed: ${response.status}`);
    }
  },

  async addComment(postId, body) {
    const response = await fetch(`/api/posts/${postId}/comments`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ body }),
    });

    if (!response.ok) {
      throw new Error(`Request failed: ${response.status}`);
    }

    return response.json();
  },
};

createPost(document.querySelector("#post"), postApi);
```

### Interview explanation

> The Like interaction is optimistic because it is reversible and benefits from immediate feedback. I save the previous state so I can roll back if the request fails. Comment text is rendered with `textContent` to avoid injecting user-provided HTML.

<a id="linkedin-post-follow-ups"></a>
### 6.2 Follow-ups

- Multiple posts with event delegation
- Comment pagination
- Nested replies
- Edit and delete
- Optimistic comments with temporary IDs
- Retry failed comments
- Like request race conditions
- Server-authoritative counts
- XSS and content sanitization
- Mentions and rich text
- Analytics
- Virtualized feed

---

<a id="calculator-ui"></a>
## 7. Priority 5: Calculator UI

This is a UI state-machine problem, separate from the plus-only string parser in the JavaScript coding round.

### Clarifying questions

> Which operators are required?

> Should calculation happen immediately or use normal precedence?

> Do I need decimals, keyboard input, or undo?

> How should division by zero be displayed?

<a id="calculator-solution"></a>
### 7.1 Interview solution

#### HTML

```html
<section id="calculator" class="calculator" aria-label="Calculator">
  <output id="display" class="display" aria-live="polite">0</output>

  <div class="keys">
    <button type="button" data-action="clear">AC</button>
    <button type="button" data-action="undo">Undo</button>
    <button type="button" data-action="sign">±</button>
    <button type="button" data-operator="/">÷</button>

    <button type="button" data-digit="7">7</button>
    <button type="button" data-digit="8">8</button>
    <button type="button" data-digit="9">9</button>
    <button type="button" data-operator="*">×</button>

    <button type="button" data-digit="4">4</button>
    <button type="button" data-digit="5">5</button>
    <button type="button" data-digit="6">6</button>
    <button type="button" data-operator="-">−</button>

    <button type="button" data-digit="1">1</button>
    <button type="button" data-digit="2">2</button>
    <button type="button" data-digit="3">3</button>
    <button type="button" data-operator="+">+</button>

    <button type="button" class="zero" data-digit="0">0</button>
    <button type="button" data-action="decimal">.</button>
    <button type="button" data-action="equals">=</button>
  </div>

  <p id="calculator-status" class="sr-only" aria-live="polite"></p>
</section>
```

#### CSS

```css
* {
  box-sizing: border-box;
}

.calculator {
  width: 320px;
  padding: 16px;
  color: white;
  font-family: system-ui, sans-serif;
  background: #1c1c1c;
  border-radius: 18px;
}

.display {
  display: block;
  min-height: 72px;
  padding: 12px;
  overflow: hidden;
  text-align: right;
  text-overflow: ellipsis;
  font-size: 2.5rem;
}

.keys {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.keys button {
  aspect-ratio: 1;
  color: white;
  background: #505050;
  border: 0;
  border-radius: 50%;
  font: inherit;
  font-size: 1.1rem;
  cursor: pointer;
}

.keys button[data-operator],
.keys button[data-action="equals"] {
  background: #ff9500;
}

.keys button[data-action="clear"],
.keys button[data-action="undo"],
.keys button[data-action="sign"] {
  color: black;
  background: #d4d4d2;
}

.keys .zero {
  grid-column: span 2;
  aspect-ratio: auto;
  border-radius: 999px;
}

.keys button:focus-visible {
  outline: 3px solid white;
  outline-offset: 2px;
}
```

#### JavaScript

```js
function createCalculator(root) {
  const display = root.querySelector("#display");
  const status = root.querySelector("#calculator-status");

  let state = {
    display: "0",    // Current display value.
    stored: null,    // Left operand.
    operator: null,  // Pending operator.
    replace: false,  // Whether the next digit replaces the display.
  };

  const history = []; // Stores snapshots for unlimited undo.

  function save() {
    history.push({ ...state }); // Store an immutable snapshot.
  }

  function render() {
    display.textContent = state.display;
  }

  function format(value) {
    if (!Number.isFinite(value)) {
      return "Error";
    }

    // Limit floating-point display noise.
    return String(Number(value.toPrecision(12)));
  }

  function compute(left, right, operator) {
    if (operator === "+") return left + right;
    if (operator === "-") return left - right;
    if (operator === "*") return left * right;
    if (operator === "/") return right === 0 ? NaN : left / right;
    return right;
  }

  function inputDigit(digit) {
    save();

    if (state.replace || state.display === "0" || state.display === "Error") {
      state.display = digit;
      state.replace = false;
    } else {
      state.display += digit;
    }
  }

  function inputDecimal() {
    save();

    if (state.replace || state.display === "Error") {
      state.display = "0.";
      state.replace = false;
    } else if (!state.display.includes(".")) {
      state.display += ".";
    }
  }

  function chooseOperator(operator) {
    save();
    const value = Number(state.display);

    if (state.operator && !state.replace) {
      state.display = format(
        compute(state.stored, value, state.operator),
      );
    }

    state.stored = Number(state.display);
    state.operator = operator;
    state.replace = true;
  }

  function equals() {
    if (!state.operator || state.replace) {
      return;
    }

    save();
    const result = compute(
      state.stored,
      Number(state.display),
      state.operator,
    );

    state.display = format(result);
    state.stored = null;
    state.operator = null;
    state.replace = true;
    status.textContent =
      state.display === "Error" ? "Calculation error." : "Result calculated.";
  }

  function clear() {
    save();
    state = {
      display: "0",
      stored: null,
      operator: null,
      replace: false,
    };
  }

  function toggleSign() {
    if (state.display === "0" || state.display === "Error") {
      return;
    }

    save();
    state.display = state.display.startsWith("-")
      ? state.display.slice(1)
      : `-${state.display}`;
  }

  function undo() {
    const previous = history.pop();

    if (previous) {
      state = previous;
      status.textContent = "Previous state restored.";
    }
  }

  function handleClick(event) {
    const button = event.target.closest("button");

    if (!button || !root.contains(button)) {
      return;
    }

    if (button.dataset.digit !== undefined) {
      inputDigit(button.dataset.digit);
    } else if (button.dataset.operator) {
      chooseOperator(button.dataset.operator);
    } else if (button.dataset.action === "decimal") {
      inputDecimal();
    } else if (button.dataset.action === "equals") {
      equals();
    } else if (button.dataset.action === "clear") {
      clear();
    } else if (button.dataset.action === "sign") {
      toggleSign();
    } else if (button.dataset.action === "undo") {
      undo();
    }

    render();
  }

  function handleKeyDown(event) {
    if (event.key >= "0" && event.key <= "9") {
      inputDigit(event.key);
    } else if ("+-*/".includes(event.key)) {
      chooseOperator(event.key);
    } else if (event.key === ".") {
      inputDecimal();
    } else if (event.key === "Enter" || event.key === "=") {
      event.preventDefault();
      equals();
    } else if (event.key === "Escape") {
      clear();
    } else if (
      (event.ctrlKey || event.metaKey) &&
      event.key.toLowerCase() === "z"
    ) {
      event.preventDefault();
      undo();
    } else {
      return;
    }

    render();
  }

  root.addEventListener("click", handleClick);
  document.addEventListener("keydown", handleKeyDown);
  render();

  return function destroy() {
    root.removeEventListener("click", handleClick);
    document.removeEventListener("keydown", handleKeyDown);
  };
}

createCalculator(document.querySelector("#calculator"));
```

<a id="calculator-undo"></a>
### 7.2 Why unlimited undo works

Before every meaningful state change:

```js
history.push({ ...state });
```

Undo restores the most recent snapshot:

```js
state = history.pop();
```

This is a simplified **memento pattern**. For a large application, consider:

- A maximum history size
- Command objects
- Patches instead of full snapshots
- Grouping related keystrokes
- Redo history
- Persisting only serializable state

### Interview explanation

> I model the calculator as a small state machine. `replace` distinguishes entering a new operand from appending digits. Before each mutation, I store a shallow snapshot, so Undo can restore any prior state without reversing every operation manually.

<a id="calculator-follow-ups"></a>
### 7.3 Follow-ups

- Operator precedence
- Repeated equals
- Percent
- Backspace
- Memory buttons
- Redo
- History size limit
- Locale-specific decimal separator
- Very large numbers
- Floating-point precision
- Extract reusable history manager

---

<a id="reported-exercises"></a>
## 8. Previously Reported LinkedIn Exercises

Practice these after the five recruiter-priority exercises.

<a id="people-you-may-know"></a>
### 8.1 People You May Know

Core requirements:

- Match a provided visual.
- Render a semantic list of profile cards.
- Load more people from an API.
- Connect or dismiss.
- Append dynamic cards.
- Use event delegation.
- Handle loading and failure.

Recommended architecture:

```js
const state = {
  people: [],
  cursor: null,
  loading: false,
  error: null,
};

function createPersonCard(person) {}
function renderPeople() {}
async function loadMore() {}
async function handleAction(action, id) {}
function handleClick(event) {}
```

Event delegation:

```js
list.addEventListener("click", (event) => {
  const button = event.target.closest("button[data-action]");

  if (!button || !list.contains(button)) {
    return;
  }

  const card = button.closest("[data-person-id]");
  handleAction(button.dataset.action, card.dataset.personId);
});
```

Important follow-ups:

- Optimistic Connect
- Undo Dismiss
- Cursor pagination
- Duplicate IDs
- Skeleton loading
- Responsive card grid
- Virtualization

<a id="tooltip"></a>
### 8.2 Tooltip

Core requirements:

- Reuse one tooltip element.
- Support many triggers.
- Use hover and keyboard focus.
- Use `role="tooltip"` and `aria-describedby`.
- Position with `getBoundingClientRect()`.
- Avoid viewport overflow.
- Hide with Escape.
- Clean up listeners.

Core positioning:

```js
function positionTooltip(trigger, tooltip) {
  const target = trigger.getBoundingClientRect();
  const tip = tooltip.getBoundingClientRect();
  const gap = 8;

  let left = target.left + target.width / 2 - tip.width / 2;
  left = Math.max(gap, Math.min(left, innerWidth - tip.width - gap));

  const top =
    target.top >= tip.height + gap
      ? target.top - tip.height - gap
      : target.bottom + gap;

  tooltip.style.left = `${left}px`;
  tooltip.style.top = `${top}px`;
}
```

> If the floating content contains interactive controls, it is a popover or dialog, not a tooltip.

<a id="top-navigation"></a>
### 8.3 Responsive Top Navigation

Core requirements:

- Semantic `<header>` and `<nav>`.
- Home logo link.
- Labeled search.
- Current-page state.
- Responsive layout.
- Keyboard-visible focus.
- Mobile menu button with `aria-expanded`.
- Cross-browser testing.

Do not make hover the only way to reveal essential navigation.

<a id="infinite-scroll"></a>
### 8.4 Infinite Scroll

Prefer a sentinel and `IntersectionObserver`:

```js
function createInfiniteScroll(sentinel, loadNextPage) {
  let loading = false;
  let hasMore = true;

  async function load() {
    if (loading || !hasMore) {
      return;
    }

    loading = true;

    try {
      const page = await loadNextPage();
      hasMore = page.hasMore;

      if (!hasMore) {
        observer.disconnect();
      }
    } finally {
      loading = false;
    }
  }

  const observer = new IntersectionObserver(
    ([entry]) => {
      if (entry?.isIntersecting) {
        load();
      }
    },
    {
      root: null,
      rootMargin: "400px 0px",
      threshold: 0,
    },
  );

  observer.observe(sentinel);

  return () => observer.disconnect();
}
```

Follow-ups:

- Scroll event plus throttle
- Retry
- Abort on teardown
- Duplicate prevention
- Cursor pagination
- “Load more” accessibility fallback
- Scroll restoration
- Virtualization

---

<a id="shared-foundations"></a>
## 9. Shared UI Foundations

### Semantic HTML

- Use `<button>` for actions.
- Use `<a>` for navigation.
- Use lists for repeated related items.
- Use headings in logical order.
- Use `<article>` for standalone post content.
- Use `<form>` and labels for user input.
- Prefer native elements before ARIA.

### CSS

Know:

- Box model and `box-sizing`
- Flexbox
- Grid
- Normal flow
- Absolute versus fixed positioning
- Overflow
- Responsive units
- Media queries
- Specificity
- `:focus-visible`
- Reduced motion
- High contrast and zoom

### JavaScript

Know:

- Event bubbling and delegation
- `target` versus `currentTarget`
- `closest()` and `contains()`
- `dataset`
- `classList`
- `hidden`
- `DocumentFragment`
- `textContent`
- Debounce and throttle
- Promises and `async`/`await`
- `AbortController`
- Stale responses
- Cleanup

### AJAX state model

Every asynchronous interaction can produce:

```text
idle
loading
success
empty
error
retry
cancelled
```

Avoid allowing two requests to pass the same guard:

```js
async function load() {
  if (loading) {
    return;
  }

  loading = true; // Set before the first await.

  try {
    await fetchData();
  } finally {
    loading = false;
  }
}
```

### Accessibility

Check:

- Accessible name
- Keyboard operation
- Visible focus
- Logical Tab order
- `aria-expanded` for disclosure
- `aria-pressed` for toggle buttons
- `aria-live` for meaningful async status
- Focus restoration after a modal
- No color-only state
- Native disabled state

### Performance

- Batch DOM insertion with a fragment.
- Avoid unnecessary full rerenders.
- Debounce text input requests.
- Throttle continuous events.
- Cancel stale work.
- Reuse shared floating elements.
- Separate infinite loading from DOM virtualization.
- Batch layout reads and writes.

### Cleanup

The code that creates an external resource should own its cleanup:

```js
function destroy() {
  controller?.abort();
  observer?.disconnect();
  clearTimeout(timer);
  root.removeEventListener("click", handleClick);
}
```

---

<a id="testing-checklist"></a>
## 10. Testing Checklist

### Every component

- Initial render
- Main interaction
- Empty input/data
- Repeated interaction
- Invalid input
- Keyboard operation
- Focus visibility
- Responsive layout
- Cleanup

### Async components

- Loading
- Success
- Empty response
- Failure
- Retry
- Slow response
- Stale response
- Duplicate action
- Component destroyed during request

### Exercise-specific

**Autocomplete**

- Query shorter than minimum
- Maximum length
- Arrow wraparound
- Enter
- Escape
- Mouse selection
- Old response arrives last

**Accordion**

- First/last item
- One-open and multi-open modes
- Nested click target
- Arrow-key focus
- Dynamic item

**Calendar**

- February in leap/non-leap years
- Month starting Sunday/Saturday
- December to January
- January to December
- Today and selected date

**LinkedIn Post**

- Like success/failure
- Rollback
- Empty comment
- Comment failure
- User text containing HTML

**Calculator**

- `0`
- Decimal
- Negative value
- Chained operations
- Divide by zero
- Clear
- Undo
- Keyboard

---

<a id="preparation-plan"></a>
## 11. Preparation Plan

### Recommended: 18 active hours over two days

#### Day 1 — Core recruiter questions, 9 hours

| Time | Task |
|---:|---|
| 45 min | Review round strategy and shared HTML/CSS/a11y |
| 2 hr 30 min | Autocomplete: study, type from memory, test |
| 1 hr 30 min | Accordion: native and custom versions |
| 2 hr | Calendar: date logic, rendering, month navigation |
| 1 hr | Repeat Autocomplete under a 60-minute limit |
| 45 min | Review mistakes and rewrite explanations |

#### Day 2 — Product UI and reported questions, 9 hours

| Time | Task |
|---:|---|
| 2 hr | LinkedIn Post: Like, comments, async failure |
| 2 hr | Calculator: state machine, keyboard, undo |
| 1 hr 15 min | People You May Know |
| 1 hr | Tooltip |
| 45 min | Navigation and Infinite Scroll review |
| 1 hr | Timed mock selected at random |
| 1 hr | Test, review, and rebuild weak sections |

### Minimum one-day plan: 10 hours

1. Autocomplete — 2.5 hours
2. Accordion — 1 hour
3. Calendar — 1.5 hours
4. LinkedIn Post — 1.5 hours
5. Calculator — 1.5 hours
6. Shared foundations — 1 hour
7. One timed mock — 1 hour

### Mastery plan: 28 hours

- Implement every priority problem twice.
- Perform five random 60-minute mocks.
- Do one version without running the code until the end.
- Explain every decision in English.
- Review all failures and maintain a mistake log.

### Practice rule

For every exercise:

1. First attempt: use the guide.
2. Second attempt: use only requirements.
3. Third attempt: 60 minutes, no guide.
4. Final review: explain trade-offs without code.

---

<a id="english-phrases"></a>
## 12. English Interview Phrases

### Clarify

> Let me restate the core user flow to make sure I understand the requirement.

> Is this action expected to update optimistically, or should I wait for the server?

> Is a real API available, or should I define a small mock boundary?

> I’ll assume the input is valid for the first version and then add validation if time permits.

### Plan

> I’ll first build the semantic structure and primary interaction. Then I’ll add asynchronous states, accessibility, and polish.

> I’m keeping state, rendering, networking, and event handling separate so each responsibility remains clear.

### Explain implementation

> I’m delegating this event from the stable container because the child items can be added dynamically.

> I’m using a native button because it already provides focus, keyboard activation, and disabled semantics.

> I’m rendering server-provided text with `textContent` so it is not interpreted as HTML.

> I set the loading flag before the first `await` so another call cannot pass the guard.

> I cancel the previous request and also compare request IDs to prevent stale results.

### Trade-offs

> This is the smallest solution that satisfies the core requirements. With more time, I would add the following production improvements.

> The advanced approach saves runtime work, but it also adds state and lifecycle complexity. I would choose it when measurements justify that trade-off.

> I’m using an optimistic update because the action is reversible. I preserve the previous state so I can roll back on failure.

### Test

> I’ll verify the normal flow first, then empty input, repeated actions, failure behavior, and keyboard interaction.

> I noticed a race condition in the current version: an older request can overwrite a newer result. I’ll fix that by cancelling the old request and checking request identity.

### Finish

> The core flow is complete. The remaining production considerations are responsive behavior, accessibility testing, analytics, and cleanup.
