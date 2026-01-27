# Project Context for AI Agents

This file documents the architecture, build system specifics, and common pitfalls for the `UI-SAMP` project to assist future AI coding agents.

## Project Structure
- `src/`
    - `main.cpp`: ASI entry point (`DllMain`).
    - `Plugin.cpp`: Main plugin logic initialization (`Load`, `Unload`, `MainLoop`).
    - `PluginGUI.cpp`: Core UI logic using ImGui. Handles drawing the scoreboard and player list.
    - `PlayerList.cpp`: Manages the list of players to display.
    - `PluginRender.cpp`: DirectX 9 hooking and rendering setup.
- `CMakeLists.txt`: Root build configuration. **Critical**: Contains manual overrides for dependencies.

## Build System (CMake) - CRITICAL
The `CMakeLists.txt` has been heavily modified to fix compatibility issues with newer CMake versions and broken dependency upstreams.
- **xbyak Override**: We use a specific commit of `xbyak` (`2a85bba...`) and a **PATCH_COMMAND** to replace its `CMakeLists.txt` because the original one has a legacy `cmake_minimum_required` that fails on modern CMake.
- **ImGui Override**: We manually define the `imgui` target and its sources (including `backends/` for DX9/Win32) because the `docking` branch (required) does not provide a standard `CMakeLists.txt` compatible with `FetchContent_MakeAvailable`.
- **PolyHook Override**: We use the `master` branch to avoid legacy CMake errors.

**Do not revert these manual overrides in `CMakeLists.txt` unless you are sure the upstream repositories have fixed their CMake configuration.**

## UI Logic (PluginGUI.cpp)
- **`drawBox`**: Renders a single player row.
    - **Cursor Management**: Uses `ImGui::Dummy` to advance the cursor layout instead of `SetCursorScreenPos`, which was causing layout issues in scroll areas.
- **`drawHeader`**: Renders the column headers (ID, Nickname, Score, Ping).
    - **Alignment**: Uses `ImGui::SetCursorPosX` before `BeginChild` to ensure the list remains centered, as `Dummy` resets the X cursor to the window/child left edge.

## Common Issues & Fixes
- **"SetCursorPos extending boundaries" Error**: Check for `SetCursorScreenPos` calls inside `BeginChild` regions. Replace them with `ImGui::Dummy(size)` if you just need to reserve space.
- **List Alignment**: If the player list shifts to the left, ensure `ImGui::SetCursorPosX` is called before `BeginChild` to center it relative to the parent window.
- **Linker Errors (LNK1181)**: If `imgui-win32.lib` or `imgui-dx9.lib` are missing, checks `CMakeLists.txt`. We should link against the main `imgui` target only, as it now bundles the backends.

## Dependencies
- **SAMP-API**: For interacting with SA-MP structures (PlayerPool, Chat, NetGame).
- **RakHook**: For hooking RakNet functions (packet sending/receiving).
- **ImGui (Docking)**: UI Library.
