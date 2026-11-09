# Applicability — ToDoList project 1

What the architecture run keeps, ignores, or must still resolve. The frozen copies in `1-1_InputDocs` stay the source. This note points at them. It does not restate them.

## Keep

These already bind the spine. They are logged as constraints and are not re-opened.

| Copy | Parts that bind |
| --- | --- |
| `briefs/brief-ToDoList-2026-10-30/brief.md` | One back end, two front ends. Google sign-in. No lists; views are computed from due dates. No offline. Do not design for scale. Deploy project 1 to production. Azure is intent only. |
| `briefs/brief-ToDoList-2026-10-30/addendum.md` | Lists, offline, and extra sign-in providers are rejected. Work arrives as sequential projects. |
| `prds/prd-ToDoList-2026-10-30/prd.md` | Glossary item shape. FR-1–FR-15, including web-only multi-move (FR-15). NFR-1 isolation, NFR-2 sign-in required, NFR-3 same views and order on both clients. Assumptions: leaving done clears the completed date; a drag across a day changes the due date and lands where it is dropped. |
| `prds/prd-ToDoList-2026-10-30/addendum.md` | Clients compute Today, week, and order from the device date. The back end stores calendar dates with no time zone. Week start is Monday, one config value behind a property. Position is per due date. |
| `ux-designs/ux-ToDoList-2026-10-30/EXPERIENCE.md` | Two surfaces and their opening screens. Signed-out behavior differs by surface. A failed server call is visible and does not discard typed text or a failed move. Web themes are swappable; that mechanism is an architecture decision. |
| `ux-designs/ux-ToDoList-2026-10-30/DESIGN.md` | Visual values are named tokens. The web look must be replaceable without hard-coded colors in behavior. |

## Ignore while drafting

Product story, voice, and pixels. They stay in the copies for context. They are not architecture decisions.

- Brief: the problem narrative, differentiation, and the “uses it most days” success story.
- PRD: journey prose and success metrics.
- Experience: voice and tone, keyboard shortcuts, swipe gestures, animation, and anti-pattern copy.
- Design: hex colors, type sizes, spacing, and component visuals.
- Not in InputDocs, and not used: memlogs, the PRD review rubric, and HTML mockups.

## Resolve in coaching

The copies name these and leave them open.

- Paradigm name for “the server stores facts; the clients compute views,” and the write boundary that keeps both clients from inventing their own save rules.
- Android framework and the API stack. .NET MAUI is intent, not a requirement. Azure hosting is intent, not a requirement.
- Document store, and whether a user’s items are embedded in one document or stored separately.
- Midnight rollover while the app is open.
- Whether “hide completed” is remembered on the device or on the server. Experience leaves this to the build.
- What “clean, readable, and commonly tested” means as a rule the back end and both clients follow.
- How production is hosted, and which environments exist before it.
