# Three AI Agents Tidy a Tablet

**English** | [繁體中文](README_zh-Hant.md) | [简体中文](README_zh-Hans.md)

**One Mac, three AI agents, one Xiaomi Pad 7 Pro: a home screen rebuilt from 4 messy pages to 2 clean ones — remotely, over wireless ADB.**

> A field report and reproducible playbook: three AI agents sharing one MacBook cooperated to clean up an Android tablet's home screen without ever touching it.

## The Cast

| Agent | Runs on | Role in this task |
|---|---|---|
| **Hermes** (Nous Research; resident AI hub on the Mac) | macOS, always-on | Orchestrator and the only "hands": wireless ADB, screen reading, gesture scripting, verification, memory |
| **ChatGPT (GPT-6, desktop app)** | macOS desktop app | Strategist: proposed the segmented-drag approach, checked the consolidation math, second opinion when the orchestrator hit walls |
| **VS Code Copilot** (headless Copilot runtime) | macOS | Execution support: code generation, script drafting, review |

The human owner stated one requirement — *"make the home screen tidy: full pages, no gaps"* — and reviewed the results. He did not assign who does what; the orchestrator did. That separation turned out to be a design principle, not a convenience (see Lessons).

## The Job

- **Target**: Xiaomi Pad 7 Pro, HyperOS 3 (Android 16)
- **Channel**: wireless ADB from the Mac (the tablet had been paired over USB once before; the pairing carried over — no cable, no re-pairing)
- **Before**: 4 home-screen pages, ~40 icons scattered across pages full of empty slots
- **After**: 2 compact pages, zero holes

## What Made It Hard

1. **Tablets have no shuffle-to-fill.** Xiaomi phones let you shake (or two-finger-pinch) to compact icons into gaps; the tablet build exposes no such gesture — we tested, nothing happened. The built-in "auto-refill after uninstall" setting only helps *future* uninstalls; it does nothing about existing holes.
2. **`adb shell input draganddrop` is too blunt** for precision work: no per-move verification, and page-crossing drags are unreliable.
3. **The fix — scripted gestures via `input motionevent`:** long-press to pick the icon up, a chain of MOVE steps, then hover against the screen edge (~0.35 s) to trigger a page flip, and release on the target cell. Page flips were confirmed in real time by watching the page-indicator dots.
4. **The floor keeps moving.** HyperOS auto-refills a gap the moment an icon leaves it, invalidating every previously measured coordinate. Iron rule: *move one icon → verify → re-read the grid → move the next.*

## The Handshake

Before the cleanup, the three agents formally introduced themselves — with programmatic proof in both directions, no human copy-paste:

- Orchestrator → Copilot via the VS Code CLI; replies read back from the Copilot runtime's `events.jsonl`.
- Orchestrator → ChatGPT desktop via clipboard + UI automation; replies read back from the macOS accessibility tree (`AXStaticText`).
- Then Copilot and ChatGPT exchanged greetings, relayed by the orchestrator — each direction verified by reading the actual text back out of the target app's window.

A side-quest worth recording: the ChatGPT desktop app has **no API**, and paste-and-Return *pretends* to work — the keystroke reports success while the message silently never lands. The reliable path is a direct write to the app's input element in the accessibility tree, followed by reading the window back to confirm. **Trust nothing; read back everything.**

## Lessons for Agent Teams

1. **Assign by capability — and let the orchestrator assign.** The user states needs and reviews results; the orchestrator decides who does what. Having the human micromanage AI division of labor defeats the point of the team.
2. **Keep one outside reviewer.** Three agents left alone converge into an echo chamber. The human's off-hand corrections ("Copilot is sitting idle", "ChatGPT can code too", "don't make me the dispatcher") improved the system three separate times.
3. **Verification culture.** "The command returned success" is not success. Read back the state you intended to change — whether it's an icon grid, a chat window, or a file.
4. **One-shot briefings.** When delegating to a code-focused agent, put file paths, input/output examples, and acceptance criteria in the *first* message. Follow-up questions burn quota and time.

## Reproduce

- A Mac with terminal **Accessibility** permission (System Settings → Privacy & Security)
- VS Code + GitHub Copilot (its headless runtime), and the ChatGPT desktop app
- An Android tablet with wireless debugging on; `adb` on the Mac
- No root, no third-party launcher, no factory reset — everything runs through standard ADB input injection and the launcher's own settings

Numbers and quotes in this README come from the actual session logs of the run.

## License

MIT
