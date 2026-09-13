# Additional BookOrbit indexer plugins

Indexer plugins for [BookOrbit](https://github.com/bookorbit/bookorbit). BookOrbit ships the loader;
these are plugins, maintained separately.

Every plugin publishes signed updates. The private Ed25519 key stays outside the repository. After
changing a plugin version, regenerate its manifest before publishing:

```sh
BOOKORBIT_PLUGIN_SIGNING_KEY=/path/to/private-key.pem node scripts/sign-update.mjs <plugin>
```

The manifest signs the exact `index.mjs` bytes. BookOrbit verifies its SHA-256 and signature before
offering or automatically installing an update.

| Plugin            | Media                   | Credential         | Grabs        |
| ----------------- | ----------------------- | ------------------ | ------------ |
| **libgen**        | ebook, comic            | none               | direct file  |
| **myanonamouse**  | ebook, audiobook, comic | session id         | torrent file |

Which sources you point BookOrbit at, and whether you are entitled to what they hold, is your
decision and your responsibility.

## Installing

Install the plugin's `index.mjs` from BookOrbit's Requests settings, or copy its directory into the
app data path and restart:

```
<APP_DATA_PATH>/plugins/indexers/<name>/index.mjs
```

`APP_DATA_PATH` is `/data` in the container, already mounted as a writable volume. Only `index.mjs`
is needed at runtime; `verify.mjs` and `fixtures/` are development files.

The plugin appears in the indexer type list under **Settings > System > Requests** and is configured
like any other indexer. Browser installs and signed updates activate immediately. Nothing is enabled
until you add a source, and a plugin that fails to load is reported at the top of that page.

## Trust

**A plugin runs inside the BookOrbit process, with that process's access:** your database, your
library files, your encryption key. Each plugin is a single dependency-free file so you can read it
before installing it.

Each plugin declares its own semantic `version` without a leading `v`. Bump it whenever that
plugin's runtime behavior changes so an installed copy can be identified from BookOrbit.

BookOrbit enforces regardless: network access only through the host (private-address policy and
per-request deadline), no claiming a built-in adapter's name, refusal of a mismatched contract
version, re-validation of resolved URLs before a download client sees them, and plugin errors
surfacing as ordinary per-indexer failures.

## The plugins

**libgen** parses search pages and resolves files through `/ads.php`. It pages through results
because other-language rows crowd out common titles on page one.

**myanonamouse** fetches a `.torrent` with the account's own session and hands it to a torrent
client. The session is ASN locked and rotates per request, so the plugin saves the rotated value
back through the host. The dynamic seedbox toggle registers the address BookOrbit connects from,
which is only correct when your download client shares it.

## Verifying

```bash
cd indexers/<name> && node verify.mjs
```

No network, no BookOrbit; exits non-zero on failure. The libgen fixtures are live pages saved byte
for byte (only a third-party contact address in the footer is replaced). The MyAnonaMouse fixtures
are written by hand, because a saved page from a private tracker would carry an account's
identifiers.

## Contract

Plugins target `PLUGIN_API_VERSION` 1. Type definitions live in `@bookorbit/plugin-api` in the
BookOrbit repository; a plugin default-exports one object declaring what it is and how to search it.
The loader refuses a version it does not speak, so a contract bump means updating both repositories
together.
