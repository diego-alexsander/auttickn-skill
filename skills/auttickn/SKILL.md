---
name: auttickn
description: Turn a feature of a web app into a narrated tutorial video with auttickn, from the app's own code. Use when the user wants a tutorial, demo or help-center video of a flow in an app whose repository you can read, or mentions auttickn. Covers onboarding the app (Convenções do app), declaring the Caminho from the code, the Validação loop, the Roteiro and the Gravação, through the auttickn MCP.
---

# auttickn: tutorial videos from the code

You read the app's repository and declare the exact path a viewer should see
(the **Caminho**). auttickn does not decide anything: it replays your Caminho
on the published app (the **Ambiente**), tells you the **first** target that
failed, and, once it passes, records the video and narrates it.

You need the auttickn MCP connected (`/mcp`, with OAuth or an organization API
key). If its tools (`list_environments`, `validate_caminho`, …) are not
available, stop and ask the user to connect it. With OAuth, request the scopes
`auttickn:mcp offline_access` (in Kiro CLI, set `"oauthScopes"` to those two;
otherwise it adds `stub` and the server answers `invalid_scope`).

## Rules

- Never ask for, read, print or send app credentials. The login is stored in the
  Ambiente by a person in the auttickn web app; you only describe the login form.
- Declare from the code. Do not guess from screenshots of other apps.
- Mark `commits: true` on the click that saves, sends, pays, publishes or
  deletes. It is not executed in a Validação, only once in the Gravação.
- Validações cost: 3 free per Tutorial, the 4th reserves the Tutorial's credit,
  10 is the hard limit. Fix the Caminho from the report before sending again.

## 1. Onboarding the app (once per app)

1. `list_environments`. Pick the Ambiente for the app (production, staging or a
   preview, published and public; no localhost, no VPN). It needs a `projectId`
   to keep Convenções. If none fits or `hasCredentials` is false for an app with
   login, ask the user to create it / store the login in the auttickn web app.
2. `get_conventions` for that project. If they exist, read them and skip to 2.
3. Otherwise discover them in the repository, and confirm with the user (the PM
   or the dev) what you could not see in the code:
   - **routes**: where the routes are defined and the URL of each screen.
   - **testAttribute**: the attribute the app uses for tests (`data-testid`,
     `data-test`, `data-cy`, stable `id`s, `name` on inputs). If none, say so:
     you will rely on `href`, `id`, `name`, placeholders and accessible names.
   - **widgets**: chat bubbles, tours, cookie banners, toasts that cover the
     screen, and how to turn them off for a recording (query param, flag).
   - **loginForm**: the login URL and how the form looks (fields, submit). Never
     the values.
   - **demoData**: what has to exist in the account for the flows to make sense,
     and what may be created by the video.
   - **notes**: anything that changes the accessible names (counters in menu
     items, translated labels), lists whose order changes, modals and drawers.
4. `save_conventions`. Each save is a new version; keep them short and factual.

## 2. Declaring a Tutorial

1. Agree with the user on the **Passos**: what the video shows, in order, in the
   user's words (one action or one screen per Passo).
2. `create_tutorial` with `featureName`, `environmentId` and `steps`.
3. Write the Caminho from the code, one action per interaction, in order:

```json
{ "op": "click", "stepIndex": 0,
  "pageUrl": "https://app.example.com/overview", "pageUrlAfter": "https://app.example.com/features",
  "fingerprint": { "role": "link", "name": "Features", "nth": 0, "selectors": ["a[href=\"/features\"]"] } }
```

- `op`: `click | type | select | hover | press | scroll`; `value` for `type`
  and `select` (realistic demo data, short).
- `stepIndex`: the Passo this action belongs to.
- `pageUrl`: the screen where it happens; `pageUrlAfter` when it navigates.
  Same base domain as the Ambiente. Getting these right matters: auttickn opens
  `pageUrl` when the screen it is on differs.
- `fingerprint.selectors`: the strongest selectors you found in the code, best
  first (test attribute, `id`, `name`, `href`). They are tried **before** the
  accessible name. Playwright selectors work (`button:has-text("Next")`,
  `>> nth=1`).
- `fingerprint.role` / `name`: the ARIA role and the accessible name a person
  sees (label, button text, `aria-label`, placeholder). `nth` only when the
  same role+name repeats on the screen.
- Never send `origin` or `effect`.

## 3. The Validação loop

1. `validate_caminho` with `tutorialId`, `commit` (the commit you read) and
   `path`. Then `get_validation` with `waitSeconds: 60` until `done`.
2. `ok` or `stopped_before_commit`: it passed; go to 4.
3. Otherwise read the report and fix **only** the action at `ref` (`p{n}` is the
   index in your `path`):
   - `not_found`: the target is not on that screen. Read `compact` (the screen
     as auttickn perceives it: roles, names, indexes) and the screenshot. Check
     `pageUrl` of this and the previous action, the selector, the name.
   - `covered`: it exists but something is on top (a menu, a modal, a toast).
     Close it in the Caminho or use the Convenções to turn the widget off.
   - `action_failed`: found, but typing/selecting failed (wrong element kind,
     option not offered).
   - `navigation_failed`: the URL did not load (wrong route, blocked).
   - `login_failed`: the stored login did not work; tell the user, it does not
     count against the Tutorial.
   - `lost` (status) or `timeout`: send the same Caminho again.
   If the cause was something the Convenções should have told you, update them
   with `save_conventions`.

## 4. Roteiro and Gravação

1. Optional but recommended: `propose_script` with a one-paragraph `summary` and
   the `intents` of each Passo (why it matters, from the code), the tone and the
   language. Review it with the user, `edit_script` (one line per Passo),
   `approve_script`. Approved means frozen.
2. `record_tutorial` with the same `commit` as the last passing Validação. The
   commits run now, once. Follow `get_job` until `completed`; give the user the
   video and article URLs.
