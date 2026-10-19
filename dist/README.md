# Loostone pinned build — IDE Index MCP Server 5.20.3

Prebuilt plugin archive used by Loostone instead of the JetBrains Marketplace build, so an
upstream Marketplace release can never silently replace the patched jar.

- **Artifact:** `jetbrains-index-mcp-plugin-5.20.3.zip` (this directory)
- **Upstream base:** `hechtcarmel/jetbrains-index-mcp-plugin` @ `03e28a4` (5.20.2)
- **Patch:** `fix: run Android run configurations on the active execution target` — the
  `ExecutionEnvironment` is built with `.activeTarget()`. Without it every Android App /
  Android Test configuration fails with `Cannot run '<name>' on '<default>'` and never
  reaches the device.
- **platformVersion:** 2025.3, `idea-version since-build="253"`
- **Built with:** Gradle 9.1.0 (wrapper pinned in this branch), JDK 21 (JetBrains Runtime 21),
  `./gradlew buildPlugin`
- **Why 5.20.3:** the plugin id stays `com.github.hechtcarmel.jetbrainsindexmcpplugin`, so the
  installed build must carry a **higher** version than anything the Marketplace offers
  (upstream latest is 5.20.2). A higher version is never offered as an update.

## Install (Android Studio / IntelliJ, from disk)

1. Close the IDE.
2. Replace the plugin directory
   `%APPDATA%\Google\AndroidStudio2026.1.4\plugins\jetbrains-index-mcp-plugin`
   with the `jetbrains-index-mcp-plugin/` directory from this zip.
3. Start the IDE and check the version under Settings → Plugins.

## Build + deploy + launch an Android app through MCP

1. A Gradle run configuration that runs `androidApp:assemble<Flavor><BuildType>` — the build step.
2. An **Android App** run configuration for the same module with the before-launch Gradle step
   disabled — the deploy + launch step (Android Studio deploys with `overlayinstall`, or falls
   back to `DeltaInstall`, then starts the activity).
3. `ide_run_tests` with `target` = that Android App configuration name.

**Pass a large `timeoutSeconds`** (e.g. 3600) for an app configuration: the run never reports
"finished", and when the timeout expires its registry **kills the app process**. Treat
`{"status":"running"}` as success — the real signal is the app in the foreground.

## Driving the server without a bound MCP client

The server speaks stateless Streamable HTTP on `127.0.0.1:<port>/index-mcp/streamable-http`
(port = the IDE process's listening port, e.g. from `netstat -ano | findstr <ide-pid>`):

```bash
curl -s -X POST "http://127.0.0.1:<port>/index-mcp/streamable-http" \
  -H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"ide_run_tests","arguments":{"target":"androidAppForAgent","waitSeconds":50,"timeoutSeconds":3600}}}'
```
