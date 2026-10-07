# AGENTS.md

### General rules
- Do not preserve backward compatibility. Remove obsolete paths instead of adding compatibility layers, fallbacks, or migrations.
- Choose the simplest implementation that fully meets the current requirements. Avoid speculative abstractions, configuration, and indirection.
- Grow the system in layers. Start from the smallest version that works end to end, and add each new capability on top of a product that already works. Never trade a working product for unfinished complexity.
- Keep components modular and concerns clearly separated.
- Prefer established, well-maintained libraries when they reduce overall complexity or improve reliability. Do not reimplement common functionality without a clear reason.
- Lean on the dependencies already in the project before writing your own implementation or adding packages. Do not assume a library lacks a capability without checking its documentation and types.
- Make architectural decisions for the long term. Do not accept a stopgap that only works for now and is meant to be replaced later.
- Never `git commit`, `git push`, or create a PR without explicit consent in the current request. Asking for an implementation ("do the fix") authorizes code and tests only; finish, verify, report, then ask before publishing.
- Never work in worktree branches. Always work in the main branch. Until asked otherwise.
- If in Plan mode or asked to plan first? Always save a plan file and tasks files in the project's local directory under ./.ai/plan for plans and ./.ai/tasks for tasks
- Always save memory related to the project in project's local directory under ./.ai/memory/{related-topic-memory}.md

### Writing style (default)
When you explain something, write about 80% of the way to ASD-STE100 Simplified Technical English:
- Keep sentences short: 20 words or fewer for instructions, 25 or fewer for descriptions.
- Give one instruction per sentence. Use the active voice and simple tenses.
- Use one word for one meaning. Do not switch synonyms for the same concept.
- Keep paragraphs to 6 sentences or fewer. Put the main point first.
- Prefer common, concrete words over jargon. Define a technical term when you first use it.
- Relax the rules when strict STE would make the text stiff or less clear. Clarity wins.

### Escalate the format when the content needs it
Pick the simplest format that makes the idea easy to understand:
1. **Prose** (STE style above): the default for most answers.
2. **Diagram**: use one when the explanation involves structure, flow, state, sequence, or relationships between parts. Prefer Mermaid in Markdown; use SVG when Mermaid cannot express it.
3. **Interactive HTML page**: use one when understanding comes from exploring: parameters to adjust, data to filter, steps to animate, or a system to poke at. Make it a single self-contained HTML file with inline CSS/JS, no build step, and openable by double-clicking.
