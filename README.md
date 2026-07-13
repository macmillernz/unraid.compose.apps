# unraid.compose.apps

App catalog for [Stacks UI](https://github.com/macmillernz/stacksUI)'s
App Store tab. Each top-level directory is one installable app.

## Adding an app

Create a new directory named after the app (this becomes its "slug" -
must match `^[a-zA-Z0-9][a-zA-Z0-9_-]{0,62}$`, the same rule Stacks UI
uses for stack names). Inside it:

- **`meta.json`** (required):
  ```json
  {
    "displayName": "Human-readable name shown on the card",
    "shortname": "the-stack-name-used-when-installed",
    "category": "A short grouping label, e.g. Media, Security",
    "description": "One or two sentences shown on the card.",
    "launchUrl": "http://[IP]:PORT/path - shown as a hint, not yet linked automatically",
    "logoUrl": "https://.../logo.png - any reachable URL, doesn't need to live in this repo"
  }
  ```
- **`docker-compose.yml`** (required): a self-contained compose file -
  don't assume any external Docker network, volume, or other resource
  already exists on the installer's system. Use `${DATA_ROOT}` (see
  below) for this app's own persistent storage rather than a hardcoded
  path.
- **`.env`** (optional): a template, not real values - every secret
  (passwords, API keys, tokens) must be an obvious placeholder the
  installer is expected to replace, never a real credential. Compose's
  `${VAR:?message}` syntax is a good way to force an app to fail loudly
  instead of silently if a required value was never filled in.

`DATA_ROOT` is special: Stacks UI's install wizard pre-fills it in the
`.env` it shows you as `<your configured data root>/<stack name>`, so a
compose file that reads its own storage paths from `${DATA_ROOT}/...`
automatically respects whatever data root the installer has configured,
instead of assuming everyone uses `/mnt/user/appdata`.

There's deliberately no per-app `README.md` - `meta.json`'s `description`
is the only thing shown, so keep it to what actually matters for
deciding whether to install.

## How this gets used

Stacks UI's App Store tab fetches this repo's top-level directory
listing plus each app's `meta.json` live, every time the tab is opened
(no caching) - to add, remove, or edit an app, just push a commit here.
Installing an app fetches that one app's `meta.json` +
`docker-compose.yml` + `.env` and opens the same New Stack wizard used
elsewhere in the plugin, pre-filled and editable before saving - nothing
is installed sight-unseen.
