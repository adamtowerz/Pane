# Project environment variables

A project can define environment variables that Pane sets in everything it starts
for that project's Panes: terminal panels, agent panels (Claude, Codex, Cursor or a
custom command), and the setup, run (Play) and archive scripts. Panes of other
projects never get them, and nothing is exported in your global shell.

Edit them in **Project Settings → Environment Variables**. Changes apply to
terminals and scripts started after you save; a terminal that is already open keeps
the environment it started with.

## Plain text or keychain

Each variable is one of:

| Source | Stored by Pane | Read when |
| --- | --- | --- |
| Plain text | The value, in the Pane database | A process starts |
| macOS Keychain, new secret | Only a reference to the item Pane creates: service `Pane`, account `<projectId>:<NAME>` | A process starts |
| macOS Keychain, existing item | Only the service and account of a generic-password item you already have | A process starts |

A keychain value is never written to the Pane database, logs, analytics, or sent to
the Project Settings window: once saved it shows as `••••••••` with a **Replace**
button. Pane runs `/usr/bin/security` to read and write items, passing a new secret
on stdin rather than the command line.

When you remove a keychain variable or delete the project, Pane deletes the item it
created for it. It never deletes an existing item you referenced.

Referencing an item another app created (for example one Conductor stored) makes
macOS ask whether `security` may read it the first time; choose **Always Allow** so
later terminals open without asking.

Keychain variables work only when the Pane host runs macOS. On another host, saving
one is refused with `KEYCHAIN_UNSUPPORTED`; use a plain value there.

## Names and precedence

Names use letters, digits and underscores and do not start with a digit. `PATH`,
`WORKTREE_PATH` and anything starting with `PANE_` are reserved: Pane always sets
them itself, so a project cannot override them. A project variable does override the
same name inherited from the environment Pane was started in.

In WSL projects the variables are added to `WSLENV`, so they reach the Linux shell.

## When a keychain item cannot be read

The terminal or script still starts, without that variable. After the shell's first
prompt, the terminal shows a `[Pane]` line naming the variable, the item and the
keychain's reason; setup and archive scripts write the same line to the Pane's logs.
The value is never printed. Fix the item in Keychain
Access (unlock the keychain, or allow access) or change the variable in Project
Settings, then open a new terminal.

## Example

A repository's `.mcp.json` reads `${UIVERIFY_API_KEY}`. Add `UIVERIFY_API_KEY` to
that project as a keychain variable (a new secret, or an existing item's service and
account), and every agent started in one of its Panes sees it, while other projects
and your login shell do not.
