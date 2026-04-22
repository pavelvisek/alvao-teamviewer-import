# Alvao Custom App: TeamViewer ID Import

Alvao Asset Management custom app that automatically syncs TeamViewer IDs to computer objects by matching hostnames against your TeamViewer tenant.

## What it does

A periodic action that pulls devices from TeamViewer (both the managed device list and the legacy one) and matches them to Alvao computer objects (desktops & notebooks) by hostname. If it finds a match, it writes the TeamViewer ID into a custom property.

When there are duplicate hostnames on the TeamViewer side, the device with the most recent `last_seen` wins.

Works in our environment, but every Alvao setup is different — **please test thoroughly before using in production.**

## ⚠️ Test before production use

**Please deploy this to a test/staging Alvao instance first** and run it with `DryRun = true` (which is the default). In DryRun mode it only logs what it *would* change without actually writing anything.

Check the Diagnostic Log, make sure the matches make sense for your environment, and only then flip `DryRun` to `false`.

If you find any bugs or unexpected behavior, please [open an issue](../../issues) — I've only been able to test this against our own setup, so feedback is very welcome.

## Requirements

- Alvao Asset Management with Custom Apps support
- TeamViewer API token with the following scopes:
  - `Computers & Contacts` — view entries
  - `Managed groups` — read operations

## Installation

1. Download `AlvaoTeamviewerImport.xml` from this repo.
2. In Alvao, go to **Administration → Asset Management → Custom Apps → Import** and import the XML.
3. Open the `Settings` script and configure at minimum:
   - `ApiToken` — your TeamViewer API bearer token
   - `ClassIdDesktop` / `ClassIdNotebook` — `lintClassId` values matching your environment
4. **Run with `DryRun = true` first.** Review the Diagnostic Log.
5. When everything looks good, set `DryRun = false`.

## Configuration

| Setting | Description | Default |
|---|---|---|
| `ApiToken` | Your TeamViewer Web API bearer token | `YOUR-API-TOKEN-HERE` |
| `ApiBase` | TeamViewer API base URL | `https://webapi.teamviewer.com/api/v1` |
| `DryRun` | `true` = only log, don't write anything | `true` |
| `EnableLogging` | `true` = info messages in Diagnostic Log (errors are always logged) | `true` |
| `RunAtHour` | Hour (0–23) to run, or `-1` for every periodic action cycle | `-1` |
| `PropertyKey` | Name of the Alvao custom property to update | `TeamViewer ID` |
| `ClassIdDesktop` | `lintClassId` for desktops in `tblNode` | `5` |
| `ClassIdNotebook` | `lintClassId` for notebooks in `tblNode` | `47` |

> **Note:** `ClassIdDesktop` and `ClassIdNotebook` values will almost certainly differ in your environment. Check your `tblNode` table or Alvao class definitions.

## How it works

1. Calls `/managed/devices` (with pagination) and `/devices` to get all TeamViewer devices.
2. Builds a hostname → TeamViewer ID map (strips domain part, case-insensitive comparison, most recent `last_seen` wins on duplicates).
3. Queries Alvao for non-discarded desktop/notebook objects that have a hostname.
4. Compares and updates the custom property where the value changed.
5. Logs a summary: updated / unchanged / not found / failed + elapsed time.

## License

[MIT](LICENSE)
