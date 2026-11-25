# Submission: Filip Bereszyński

## Time Spent

Total time: **04 hours 24 minutes** (approximate)

## Ticket Triage

### Tickets I Addressed

List the ticket numbers you worked on, in the order you addressed them:

1. **CFG-142**: Fixed a race condition in price calculation by introducing sequential request IDs and ensuring that only the latest request can update the price, error, and loading states.

2. **CFG-143**: Fixed the missing cleanup of the window resize event listener by removing the listener when the component unmounts.

3. **CFG-144 and CFG-145**: Identified conflicting requirements regarding the Quick Add feature. CFG-144 suggests removing the feature, while CFG-145 proposes adding keyboard shortcut support. Since the TechStyle demo requirements indicate that Quick Add is needed, I postponed both changes and requested clarification before proceeding.

4. **CFG-148**: Fixed a critical bug in add-on dependency handling. Replaced direct array mutation with immutable state updates and corrected the dependency cleanup logic to handle both select and toggle options. This ensures that dependent add-ons are automatically removed when their requirements are no longer met, preventing application crashes.

5. **CFG-147**: Fixed Unicode encoding issues and made configuration sharing URLs URL-safe. Replaced direct Base64 encoding with UTF-8 encoding using TextEncoder and TextDecoder, and implemented URL-safe Base64 conversion. The decoder also maintains compatibility with legacy standard Base64 links. Verified Unicode round-trip, URL-safe output, and legacy Base64 decoding.

6. **CFG-158**: Reviewed the reported performance issue and inspected the ProductConfigurator component and its parent (App.tsx). Found that currentConfig is already memoized using useMemo, while product is passed as a stable imported mockProduct reference. The callback props are also memoized using useCallback. No additional memoization was introduced because the suggested changes were already partially implemented, and further optimizations without profiling evidence could add unnecessary complexity.

7. **CFG-152**: Implemented keyboard accessibility improvements across the product configurator, addressing the reported issues and additional accessibility problems discovered during code review.
   Changes made in CFG-152:

- Color picker: Implemented keyboard navigation using arrow keys, Home, End, Enter, and Space. Added roving tabIndex so only the selected (or first available) color is included in the normal tab order. Unavailable colors are skipped, and read-only mode prevents selection changes.
- Focus indicators: Added visible :focus-visible outlines to color swatches, quantity controls, and primary buttons to make keyboard focus easier to identify.
- Modal focus management: Added automatic focus transfer into the modal when opened and focus restoration to the original trigger after closing. Implemented a focus trap to keep keyboard navigation inside the modal and preserved Escape key handling.
- Modal semantics: Added role="dialog", aria-modal="true", and aria-labelledby to both modals to provide appropriate information to assistive technologies.
- Add-on checkboxes: Fixed the empty onChange handler so checkboxes can be operated using the keyboard. Prevented event propagation to avoid triggering the parent click handler and toggling an add-on twice.
- Toggle switches: Improved keyboard interaction and ensured that read-only switches cannot be changed using Enter or Space. 8. **CFG-146**: Fixed the draft "Last saved" timestamp handling by using the actual savedAt value returned from saveDraft() instead of creating a new timestamp independently. The ISO timestamp is parsed into a JavaScript Date object and displayed using the user's local timezone. Verified that the displayed time matches the expected local time after saving a draft. 9. **CFG-149**: Improved the price calculation loading state by adding animated loading dots next to the old price label. The existing opacity change was preserved to make it clear that the displayed price is being recalculated. Added support for reduced-motion accessibility preferences. Verified that the loading indicator appears during price calculation and disappears when the calculation completes. 10. **CFG-150**: Fixed the color picker layout by replacing the flex-based layout with CSS Grid and using the dynamically calculated column count. Updated the resize handler to calculate and apply the column count independently to every color picker instead of relying on a single shared ref. This ensures that color swatches are arranged consistently across different viewport widths.

### Tickets That Need Clarification

List any tickets where you couldn't proceed due to ambiguity:

| Ticket              | Question                                                                                                                                                                                                                                                                                                  |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CFG-144 and CFG-145 | Should the Quick Add feature remain in the product? CFG-144 suggests removing it, while CFG-145 proposes adding keyboard shortcut support. Since the TechStyle demo requirements indicate that Quick Add is needed, should I keep the feature and implement the keyboard shortcut, or remove it entirely? |

### Tickets With a Misleading Priority Label

Our board labels aren't always right. List tickets where you disagreed with the label (e.g. a Low that's actually High, a High that's actually Low, a Medium that's actually Unclear) and explain why. If you didn't disagree with any, say so.

| Ticket  | Label on board | Your read    | Why                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------- | -------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CFG-152 | e.g. ⚪ Low    | e.g. 🟠 High | The issue affects users who rely on keyboard navigation and prevents them from fully using the product configurator. More importantly, the TechStyle MSA includes accessibility compliance requirements, making this more than a minor usability issue. Leaving these accessibility barriers unresolved could put the upcoming client demo and contractual compliance at risk. For this reason, I consider this issue High priority rather than Low. |

### Tickets Where You Stopped to Ask Instead of Implementing

If a ticket was ambiguous enough that the right move was to escalate / ask PM / gather more context rather than ship a fix, list it here. We value this signal — picking one interpretation blindly and shipping it can cost more than stopping to ask.

| Ticket              | Who would you ask | What you'd ask                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| CFG-144 and CFG-145 | e.g. PM           | I found a conflict between CFG-144 and CFG-145. CFG-144 suggests removing Quick Add, while CFG-145 proposes adding keyboard shortcut support for it. However, the TechStyle requirements indicate that Quick Add is needed for the upcoming demo in two weeks. Could you clarify whether Quick Add should remain part of the product? Should I proceed with the keyboard shortcut implementation, or should the feature be removed? I have intentionally postponed changes to both tickets until this is clarified to avoid implementing conflicting requirements. |

---

## Technical Write-Up

### Critical Issues Found

Describe the most important bugs you identified:

#### Issue 1: Dependent Add-ons Causing Application Crash

**Ticket(s):** CFG-148

**What was the bug?**

When a user changed an option that an add-on depended on, the application did not always correctly remove the now-unavailable add-on from the selected add-ons state.

When a user changed an option that an add-on depended on, the application did not always correctly remove the now-unavailable add-on from the selected add-ons state.

When a user changed an option that an add-on depended on, the application did not always correctly remove the now-unavailable add-on from the selected add-ons state.

**How did you find it?**

I inspected the option change handler, add-on selection logic, and summary rendering code. I traced how dependent add-ons were handled when their parent option changed and identified the direct state mutation and unsafe lookup.

I then reproduced the issue by selecting a dependent add-on and changing the option that controlled its availability.

**How did you fix it?**

Replaced direct array mutation with immutable state updates using filter() and a functional state setter.

Updated the dependency cleanup logic to handle both select and toggle options, ensuring that dependent add-ons are automatically removed when their requirements are no longer met.

**Why this approach?**

Immutable state updates follow React's state management principles and ensure that React can correctly detect changes through reference comparison.

I chose to fix the dependency cleanup at the point where the parent option changes rather than introducing additional defensive logic throughout the application. This keeps the behavior predictable and prevents invalid add-on selections from reaching the summary rendering logic.

---

#### Issue 2: Race Condition in Price Calculation

**Ticket(s):** CFG-142

**What was the bug?**

Multiple price calculation requests could run simultaneously when users changed product options quickly.

Because the application did not correctly identify which request was the latest, an older response could overwrite a newer price. Additionally, an older request could incorrectly update the loading state while a newer request was still running.

The root cause was incorrect request identification: the implementation compared Date.now() with a response timestamp that was actually a request counter. The finally block also updated the loading state without checking whether the request was still the latest one.

**How did you find it?**

Because the application did not correctly identify which request was the latest, an older response could overwrite a newer price. Additionally, an older request could incorrectly update the loading state while a newer request was still running.

The root cause was incorrect request identification: the implementation compared Date.now() with a response timestamp that was actually a request counter. The finally block also updated the loading state without checking whether the request was still the latest one.

**How did you fix it?**

Because the application did not correctly identify which request was the latest, an older response could overwrite a newer price. Additionally, an older request could incorrectly update the loading state while a newer request was still running.

The root cause was incorrect request identification: the implementation compared Date.now() with a response timestamp that was actually a request counter. The finally block also updated the loading state without checking whether the request was still the latest one.

**Why this approach?**

Using a request ID provides a straightforward way to distinguish between current and outdated requests without introducing additional dependencies or unnecessary complexity.

I considered the fact that simply ignoring outdated responses is not sufficient: outdated requests must also be prevented from changing the loading state. Therefore, the same latest-request check is applied consistently across the request lifecycle.

---

#### Issue 3: Keyboard Accessibility Barriers

**Ticket(s):** CFG-152

**What was the bug?**

Several configurator controls could not be fully operated using a keyboard alone.

The color picker used non-focusable div elements with radio roles but lacked keyboard navigation. Modal dialogs did not correctly manage focus, and add-on checkboxes had an empty onChange handler, preventing proper keyboard interaction.

Toggle switches also had incomplete read-only handling, allowing keyboard input to change their values even when the configurator was in read-only mode.

**How did you find it?**

Several configurator controls could not be fully operated using a keyboard alone.

The color picker used non-focusable div elements with radio roles but lacked keyboard navigation. Modal dialogs did not correctly manage focus, and add-on checkboxes had an empty onChange handler, preventing proper keyboard interaction.

Toggle switches also had incomplete read-only handling, allowing keyboard input to change their values even when the configurator was in read-only mode.

**How did you fix it?**

Several configurator controls could not be fully operated using a keyboard alone.

The color picker used non-focusable div elements with radio roles but lacked keyboard navigation. Modal dialogs did not correctly manage focus, and add-on checkboxes had an empty onChange handler, preventing proper keyboard interaction.

Toggle switches also had incomplete read-only handling, allowing keyboard input to change their values even when the configurator was in read-only mode.

**Why this approach?**

Several configurator controls could not be fully operated using a keyboard alone.

The color picker used non-focusable div elements with radio roles but lacked keyboard navigation. Modal dialogs did not correctly manage focus, and add-on checkboxes had an empty onChange handler, preventing proper keyboard interaction.

Toggle switches also had incomplete read-only handling, allowing keyboard input to change their values even when the configurator was in read-only mode.

### Other Changes Made

Brief description of any other modifications:

**CFG-143** — Resize Event Listener Cleanup
Fixed the missing cleanup of the window resize event listener by removing it when the component unmounts. This prevents unnecessary event listener accumulation and potential memory leaks.

**CFG-146** — Last Saved Timestamp
Fixed the draft timestamp handling by using the actual savedAt value returned from the API instead of generating a separate timestamp on the client. The timestamp is parsed and displayed in the user's local timezone.

**CFG-147** — Unicode Encoding and URL-safe Base64
Implemented UTF-8 encoding and decoding using TextEncoder and TextDecoder to support Unicode characters in shared configurations. Also introduced URL-safe Base64 encoding while maintaining compatibility with existing standard Base64 links.

**CFG-149** — Price Calculation Loading Indicator
Improved the price calculation loading state by adding animated loading dots next to the Total Price label while preserving the existing opacity indicator. Also added support for reduced-motion accessibility preferences.

**CFG-150** — Color Picker Layout
Fixed the color picker layout by replacing the flex-based layout with CSS Grid and applying dynamically calculated column counts. Updated the resize handler to calculate the layout independently for each color picker.
Performance Analysis

**CFG-158** — ProductConfigurator Performance Optimization
Reviewed the reported performance issue and inspected the ProductConfigurator component and its parent. Found that currentConfig was already memoized using useMemo, while the product reference and callback props were stable; no additional memoization was introduced without profiling evidence to avoid unnecessary complexity.
Tickets That Need Clarification

**CFG-144 and CFG-145** — Quick Add Feature Conflict
Identified conflicting requirements: CFG-144 suggests removing Quick Add, while CFG-145 proposes adding keyboard shortcut support. Since the TechStyle demo requirements indicate that Quick Add is needed, implementation was postponed pending clarification from the Product Manager to avoid making conflicting changes.

---

## Code Quality Notes

### Potential Improvements for the Future

If you had more time, what would you improve?

1. Add automated unit and integration tests for critical functionality, especially price calculations, dependent add-ons, and keyboard accessibility.
2. Perform a more comprehensive WCAG 2.1 AA accessibility audit and use React DevTools Profiler to identify further performance improvements.

---

## Questions for the Team

Questions you would ask in a real scenario:

1. Could you clarify the conflicting requirements in CFG-144 and CFG-145 regarding the Quick Add feature?
2. Are there existing automated tests or accessibility standards that should be followed when implementing future changes?

---

## Assumptions Made

List any assumptions you made to proceed:

1. The existing API behavior and data structures should remain unchanged unless a ticket explicitly requires modifications.
2. Native HTML elements, such as "select", should be preserved when they already provide the required functionality and accessibility.

---

## Self-Assessment

### What went well?

I successfully identified and fixed several important bugs, carefully investigated their root causes, and tested the implemented solutions. I also made sure to understand existing code before introducing changes.

### What was challenging?

Some issues required tracing interactions between multiple components and keyboard accessibility.

### What would you do differently with more time?

I would investigate further performance improvements using React DevTools Profiler for the whole project or dedicated tasks.

---

## AI Assistance Notes (optional)

If you used AI tools (Copilot, Claude, Cursor, ChatGPT, etc.), we'd love a short note on how. **We don't score the fact that you used AI — we score the quality of your reasoning and verification.**

Useful things to share:

- What kinds of prompts you sent (paraphrase is fine, no need to paste logs)
- Where the AI helped and where you overrode it
- Anything the AI suggested that you rejected — and why
- Anything you verified manually that the AI got wrong

I used AI as a supporting tool to discuss potential solutions, clarify technical concepts, and review a few implementation decisions.

The majority of the work involved my own code analysis, debugging, implementation, and manual testing. I evaluated the suggestions critically and verified the results myself rather than relying on AI-generated solutions without testing.

For example, I used ChatGPT AI model to discuss possible approaches to handling race conditions in asynchronous price calculations (CFG-142). I used the discussion to validate my understanding of request handling.

---

## Additional Notes

Anything else you want us to know:

In my work on the tasks, I prioritized keeping changes maintainable, and consistent with the existing codebase. I also focused on understanding the root cause of each issue.
