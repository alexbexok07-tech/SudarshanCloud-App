# SudarshanCloud Android App

SudarshanCloud is an Android client for managing servers hosted by a
Pterodactyl panel. It connects directly to the panel API using the API keys
you configure in the app. It does not contain hard-coded demo servers.

This README covers:

- Installing and configuring the app
- What each screen and menu item does
- Client and Application API key permissions
- Monitoring, notifications, and refresh behavior
- Building the project from source
- Troubleshooting common problems
- Where the important files are located
- Support and help tickets

## Support

For help, open a ticket in the SudarshanCloud Discord server:

https://discord.gg/9jvSQPcKEc

When opening a ticket, include:

1. The screen where the problem occurred
2. The exact error message
3. Whether the problem affects one server or every server
4. Your Android version
5. The app version or APK build you installed
6. A screenshot with API keys, passwords, and private server details hidden

Never post a Client API key, Application API key, password, session cookie, or
other credential in Discord, GitHub Issues, screenshots, or build logs.

## Quick start

### 1. Install the APK

Install the debug APK produced by the GitHub Actions workflow or by a local
Gradle build. Android may ask you to allow installation from the browser or
file manager that opened the APK.

The project supports Android API 24 and newer. The build targets Android API
36.

### 2. Open Settings

Open the navigation drawer with the menu button, then select **Settings**.
Settings is also available from the top-right gear icon and the bottom
navigation bar.

### 3. Enter the panel URL

Use the URL of the Pterodactyl panel, for example:

    https://portal.sudarshancloud.com

Use the panel URL itself, not a server URL and not a URL ending in an API
path. The app adds the required API paths automatically.

### 4. Add a Client API key

Create a Client API key in the Pterodactyl panel and paste the value beginning
with `ptlc_` into **Client API Key**.

The Client API key is the normal key used for servers that belong to the
authenticated client account. It is used for server resources, power actions,
console access, files, backups, databases, startup variables, and schedules
when the panel permits those operations.

### 5. Optional: add an Application API key

If you administer the panel and want the app to display servers belonging to
other panel users, add an Application API key beginning with `ptla_`.

Enable **See other panel servers** after entering the Application API key.
The app then loads the complete panel server inventory through the
Application API. It still tries the configured Client API key for client-level
actions.

Only administrators should use an Application API key. An Application API key
has broader permissions than a normal Client API key.

### 6. Optional: enter an account email

The account email is displayed in the navigation drawer. It is not a
replacement for an API key and is not required for API requests.

### 7. Save and sync

Tap **Save & Sync**. The app saves the configuration locally, updates its API
client, and requests the live server list.

If the server list is empty, check the panel URL and API key permissions before
trying to use the management screens.

## Authentication and credential storage

The **Authentication** destination provides a separate credential entry flow.
It accepts:

- Pterodactyl Panel URL
- Client API key
- Optional account email

The app describes these credentials as being protected with Android Keystore
and EncryptedSharedPreferences using AES-256-GCM. Do not treat the app as a
place to share credentials: keep keys private and rotate them in the panel if
you think they were exposed.

The **Settings** screen is the main configuration screen for normal use because
it also exposes the Application API key, the all-servers switch, and the
background monitor switch.

## App navigation

The app has two main navigation areas:

- The top-left menu opens the full sidebar.
- The bottom navigation bar provides quick access to Servers and Settings.

The current sidebar destinations are described below.

### Servers

**Servers** is the home screen and the source for selecting a server.

It provides:

- Live servers returned by the configured Pterodactyl API
- Server search
- Server name, node, address, port, and status
- CPU, memory, and disk summaries when live telemetry is available
- Start, restart, and stop controls
- Navigation into the selected server dashboard

The server list is synchronized from the panel. A successful empty response is
treated as an empty inventory; old demo or stale cards are not retained as
fake servers.

The app refreshes resource telemetry approximately every three seconds while
the main app is open. Requests for different servers are run concurrently,
while each server's own request remains controlled so requests do not overlap
unnecessarily.

### Dashboard

The **Dashboard** opens after selecting a server from the Servers screen.

It shows:

- Server name
- Node and primary address
- Current status
- Uptime
- Network receive and transmit totals
- Start, restart, and stop controls
- Expandable CPU, memory, and disk statistics
- Node address and connection details
- SFTP host and port
- Server identifier
- Quick links to Remote Console, File Manager, Backups, and Databases

Tap a connection detail to copy it to the Android clipboard.

If the server is discovered through the Application API but is not accessible
through the client account, the dashboard marks it as an “Other panel server”.
Actions then depend on the configured API keys and the permissions granted by
the panel.

### Remote Console

**Console** connects to the server console using the panel's WebSocket
credentials. The app also has an HTTP command fallback when a live WebSocket
connection is not available.

Use it to:

- Read console output
- Send commands
- Clear the visible local console log
- Use the power controls

Console access requires the panel to permit console commands for the selected
server. A server can be online while console access is denied by API
permissions.

### File Manager

**Files** lets you work with the selected server's filesystem through the
Pterodactyl file API.

Available operations include:

- Browse directories
- Open files
- Edit file text
- Save file changes
- Create folders
- Create files
- Delete files

Use care with delete and save actions. The app does not create a separate
version history for files.

### Backups

**Backups** lists the selected server's panel backups and can:

- Refresh the backup list
- Create a backup
- Delete a backup

Backup availability, retention, and creation permissions are controlled by
the panel and the server's plan.

### Databases

**Databases** lists databases attached to the selected server and can request a
new database when the panel allows it.

Creating a database may require:

- A database name
- A numeric database host ID
- A database host configured in the Pterodactyl panel
- Permission for the API key to create databases

If database creation is not enabled for the account, create the database from
the panel web app instead and then use the Android app to view it.

### World Manager

**World Manager** finds Minecraft world folders through the file API. Select a
world to open its directory in the File Manager.

The world list depends on the server's filesystem layout and the permissions
of the configured API key. The app does not assume that every server uses the
same world folder names.

### Startup / Version

**Startup / Version** loads the server's startup variables. It can display and
update values such as the selected version or other panel-defined startup
settings.

Only variables exposed by the panel are shown. Defaults are used for display
when the server does not currently have a custom value. Changes are sent back
to the panel, so verify a value before saving it.

### Schedules

**Schedules** provides the server's scheduled task list and can:

- Refresh schedules
- Create a schedule
- Execute a schedule
- Delete a schedule

The exact command, timing, and allowed operations are determined by the panel.
Use the panel web interface if a schedule needs advanced editing that is not
available in the Android screen.

### Authentication

**Authentication** is the credential entry screen described in the
authentication section above. Use it when you need to enter or re-enter the
panel URL and Client API key.

After successful authentication, the app returns to the Servers screen and
syncs the live inventory.

### Settings

**Settings** contains:

- Panel Endpoint
- Client API Key
- Application API Key
- See other panel servers
- Account Email
- Save & Sync
- Reset
- Background State Monitor

The **Reset** button restores the panel URL field to the default
SudarshanCloud portal URL. It does not automatically erase every stored
credential unless the surrounding app flow explicitly replaces and saves
those fields.

## Power actions and HTTP 500 behavior

The dashboard and console provide:

- Start
- Restart
- Stop

If a restart request receives HTTP 500 from the panel or Wings, the app tries
the equivalent sequence:

1. Send stop
2. Wait briefly
3. Send start

This fallback is limited to restart responses with HTTP 500. Other HTTP
errors are not silently converted into stop/start actions.

The app extracts useful detail from panel error responses when the response
contains it. A 401 or 403 normally means the key is invalid or lacks the
required permission. A 404 normally means the panel endpoint or server
identifier was not found.

## Resource monitoring and notifications

### In-app telemetry

The main app requests server telemetry on an approximately three-second
cadence. CPU, memory, disk, status, uptime, and network totals are updated
when the panel returns live statistics.

When live statistics are temporarily unavailable, the app keeps the last
known server metadata and reports that live telemetry is unavailable instead
of inventing a new value.

### Background State Monitor

Settings includes a **Background State Monitor** switch. When enabled, the
foreground data-sync service polls server nodes every ten seconds and sends a
notification when it detects a START, STOP, or CRASH state change.

Android may require notification permission. On Android 13 and newer, allow
notifications when the app asks. Battery optimization or background execution
restrictions can delay notifications.

Disable the switch to stop the background monitor. The monitor is separate
from the approximately three-second in-app telemetry refresh.

## API key permissions

The app can only do what the Pterodactyl panel grants to the configured key.
If a feature is visible but fails, check the key permissions first.

### Client API key

Use a Client API key for normal server operations:

- View client-accessible servers
- View resources
- Use power actions
- Use console access
- Read and edit files
- Read and manage backups
- Read and create databases
- Read and update startup variables
- Read and manage schedules

The panel may expose fewer permissions depending on the key and account.

### Application API key

Use an Application API key for panel-wide server discovery. It is mainly
needed for **See other panel servers** and should be restricted to trusted
administrators.

Do not paste either key into a support ticket. If a key is exposed, revoke it
in the panel and create a replacement.

## Troubleshooting

### The app shows no servers

1. Open Settings.
2. Confirm the panel URL is reachable in a browser.
3. Confirm the Client API key starts with `ptlc_`.
4. Confirm the key belongs to the expected panel account.
5. Tap **Save & Sync**.
6. If using all-server discovery, confirm the Application API key starts with
   `ptla_` and enable **See other panel servers**.
7. Check that the panel returns servers for that account.

### “API key was rejected” or HTTP 401/403

The key is invalid, expired, revoked, or missing a permission. Create a new
key in the panel and save it again. Use a Client API key for client endpoints
and an Application API key for panel-wide inventory.

### HTTP 404

Check that:

- The panel URL is the base panel URL
- The server still exists
- The server identifier is correct
- The selected server was not deleted or moved

Use **Save & Sync** to replace the local server inventory with the current
panel inventory.

### Restart still fails

The app already tries stop-then-start when restart returns HTTP 500. If the
fallback also fails:

- Check that the key can stop and start the server
- Check the server's Wings/node health in the panel
- Check whether the server is suspended
- Try the same action in the Pterodactyl web panel
- Record the exact error detail shown by the app for a support ticket

### Node or host information is unavailable

The app prefers live allocation and node data. If the live response omits
those values, it preserves usable cached metadata from the previous sync. It
does not treat placeholders such as “Unknown host” or “Unknown node” as real
addresses.

Run **Save & Sync** after correcting allocations or node data in the panel.
If the panel itself does not return an address, the app cannot invent one.

### Telemetry is missing or not updating

- Confirm the server is selected.
- Confirm the Client API key can read resources.
- Confirm the server is online or in a state where the panel returns stats.
- Pull to the app's refresh action or use the top-bar refresh button.
- Wait through at least one three-second polling cycle.
- Check whether the panel or Wings is returning an error.

The background monitor's ten-second interval is separate from the in-app
three-second telemetry interval.

### Console will not connect

Check the Client API key's console permission and the panel's WebSocket
availability. A server can have working file access while console access is
disabled. Try sending a simple command only after the console status shows a
connection.

### File changes fail

Check file permissions, path spelling, server state, and whether the target
file is locked or managed by a server process. The File Manager uses the
Pterodactyl API and cannot bypass panel permissions.

### Database creation fails

The panel must have a database host configured and the API key must have
database creation permission. Confirm the numeric database host ID in the
panel web app. If the account is not allowed to create databases, create one
in the web app instead.

### Background notifications do not arrive

- Confirm Background State Monitor is enabled.
- Allow Android notifications.
- Remove battery restrictions for the app if the device pauses background
  work.
- Confirm the server has a usable node address.
- Remember that the background monitor polls every ten seconds and only
  reports detected state changes.

### A GitHub build log shows an AWT or IntelliJ null-pointer message

The message involving
`ApplicationManager.getApplication()` can come from Java/IDE tooling used
while Gradle tasks are running. If later Gradle tasks continue and the job
produces an APK, it is not automatically an app runtime crash.

Look for the first actual `FAILURE`, `Execution failed`, Kotlin compiler
error, or missing artifact after that message. If the workflow completes and
the APK is downloadable, install and test the APK normally. If the workflow
stops, include the first real failure section in a Discord ticket.

### The APK will not install

- Confirm the download completed.
- Uninstall an older build if Android reports a signing conflict.
- Allow installation from the app used to open the APK.
- Make sure the device runs Android API 24 or newer.
- Download the APK artifact again if the file is incomplete.

## Building from source

### Requirements

For a local Android build, install:

- Java 21 or newer
- Android SDK with Android API 36
- Android build tools compatible with the Android Gradle Plugin
- A network connection for Gradle dependencies on the first build

The Gradle wrapper uses Gradle 9.3.1. The project compiles the app with Java
11 source/target compatibility, but the Gradle runtime itself must be able to
run on the Java version required by the wrapper and plugins.

### Run tests

From the project root:

    ./gradlew test --no-daemon

### Build a debug APK

    ./gradlew assembleDebug --no-daemon

The debug APK is normally written under:

    app/build/outputs/apk/debug/

### Build on GitHub Actions

The workflow is located at:

    .github/workflows/build-apk.yml

To use it:

1. Create or open a GitHub repository.
2. Upload the project files while preserving the `.github/workflows` folder.
3. Open the repository's **Actions** tab.
4. Choose **Build SudarshanCloud APK**.
5. Select **Run workflow**.
6. Download the `SudarshanCloud-debug-APK` artifact after a successful run.

More concise build notes are in `BUILD_ON_GITHUB.md`.

### Release signing

The release build is configured to use a keystore path and passwords supplied
through environment variables:

- `KEYSTORE_PATH`
- `STORE_PASSWORD`
- `KEY_PASSWORD`

The configured release alias is `upload`. Never commit the keystore or any
password to the repository. Use GitHub Actions secrets or another protected
secret store.

### Optional environment files

`.env.example` documents the optional `GEMINI_API_KEY` convention used by the
Secrets Gradle Plugin. Do not put a real API key into a committed `.env` file.
Only configure it when the related feature or build environment requires it.

## Where to find what

### User-facing app code

- `app/src/main/java/com/sudarshancloud/app/MainActivity.kt`
  - App entry point
  - Drawer, top bar, bottom navigation, and destination routing
- `app/src/main/java/com/sudarshancloud/app/ui/components/AppDrawer.kt`
  - Sidebar menu and the footer information/link
- `app/src/main/java/com/sudarshancloud/app/ui/screens/`
  - Main screen implementations
- `app/src/main/java/com/sudarshancloud/app/ui/viewmodel/`
  - Screen state, refresh actions, navigation, and user actions
- `app/src/main/java/com/sudarshancloud/app/ui/theme/`
  - Colors, typography, and Compose theme

### Panel and server data

- `app/src/main/java/com/sudarshancloud/app/data/api/`
  - Retrofit API definitions, API client, and console WebSocket support
- `app/src/main/java/com/sudarshancloud/app/data/model/`
  - Pterodactyl response and request models
- `app/src/main/java/com/sudarshancloud/app/data/repository/ServerRepository.kt`
  - Panel synchronization, telemetry, power actions, console, files,
    backups, databases, startup variables, schedules, and API errors
- `app/src/main/java/com/sudarshancloud/app/data/db/`
  - Room database, local configuration, cached servers, folders, and command
    history

### Background monitoring

- `app/src/main/java/com/sudarshancloud/app/service/ServerStateMonitorService.kt`
  - Foreground background-state monitor and notifications
- `app/src/main/AndroidManifest.xml`
  - Internet, network-state, notification, foreground-service, and launcher
    declarations

### Build and dependency configuration

- `settings.gradle.kts`
  - Gradle project settings
- `build.gradle.kts`
  - Root Gradle configuration
- `app/build.gradle.kts`
  - Android application, SDK levels, build types, and dependencies
- `gradle/libs.versions.toml`
  - Central dependency and plugin versions
- `gradle/wrapper/gradle-wrapper.properties`
  - Gradle wrapper version; currently Gradle 9.3.1
- `.github/workflows/build-apk.yml`
  - GitHub Actions APK build workflow
- `BUILD_ON_GITHUB.md`
  - Short GitHub Actions build guide

## Data and privacy notes

- API credentials are entered by the user and should be treated as secrets.
- The app needs network access to communicate with the panel.
- Local cached server metadata is used to keep the interface useful when a
  later live response omits node or host details.
- Live panel data remains authoritative during a successful synchronization.
- Revoke and replace API keys if a device is lost or a key is exposed.

## Project contacts shown in the app

- Website Portal: https://portal.sudarshancloud.com
- App developer: @alexbexok.com
- App tester: @yourlocal_pragbaler_73733
- Helper: aayushmc0
- Built using: Gradle 9.3.1

Copyright © 2026 SudarshanCloud. All rights reserved.