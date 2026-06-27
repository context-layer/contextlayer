# Context Layer

A VS Code extension that holds your current development decisions and re-surfaces them to Copilot, so the model stops drifting back to superseded choices.

---

## The problem

You tell Copilot to use Tailwind. Three steps later it's writing custom CSS again. You correct it. It drifts back. You're spending more time re-stating decisions than writing code.

The model isn't broken. It just doesn't hold your corrections across turns.

---

## What Context Layer does

You maintain a list of current decisions, things you've decided for this session that the model keeps forgetting. Context Layer holds that list and keeps it in front of Copilot on every turn, invisibly.

You add, edit, or delete decisions as your session evolves. When you change your mind, you replace the old decision. The model never sees the superseded version.

---

## Install

[Download the VS Code extension](https://github.com/context-layer/contextlayer/releases/download/v0.0.1/context-layer-0.0.1.vsix)

1. In VS Code: `Cmd+Shift+P` → "Install from VSIX" → select the downloaded file
2. Run `CL: Set Invite Token` from the command palette and enter your token
3. Open a workspace folder

To get an invite token, contact [contextlayerkaisek@gmail.com](mailto:contextlayerkaisek@gmail.com).

---

## Usage

- Open the **Context Layer** sidebar (list icon in the activity bar)
- Add your current decisions (for example, use Tailwind, no external API calls, TypeScript only, whatever governs this session)
- Use `@cl` in Copilot Chat for turns where those decisions matter
- Add, edit, or delete decisions as your session evolves

---

## Status

v0.0.1. Early access. Invite only.
