# MiniIDA

**MiniIDA** is an Android application for static binary analysis. It provides a disassembler-style interface, heuristic pseudo-C output, control-flow visualization, and an integrated **MCP server**.

> **Sorry:** MiniIDA's application interface is currently available in French only. An English interface is planned, and I will work on adding it in a future update.

## Features

### Binary analysis
- Open ELF files (`.so`, ELF executables) and raw binary files.
- Inspect ELF class, type, architecture, entry point, sections, segments, and dynamic dependencies.
- Display SONAME and Build ID when available.
- Inspect detected security properties, including PIE/PIC, NX, RELRO, Stack Canary, and Fortify.
- Inspect imports, exports, local symbols, and relocations.
- Identify `JNI_OnLoad` and `JNI_OnUnload` when present.
- Inspect initialization and finalization arrays (`init_array` / `fini_array`).

### Disassembly and navigation
- Interactive disassembly powered by Capstone.js and WebAssembly.
- Architectures offered by the interface: ARM64, ARM, Thumb, x86, and x86-64.
- Navigate by address or symbol name.
- Browse backward and forward through navigation history.
- Search functions, symbols, and strings.
- Search byte patterns with wildcards (`??`), ASCII/UTF-16 text, and immediate values.
- View bytes and strings associated with analyzed data.
- Identify the section and permissions associated with the current address.

### Pseudo-code and control-flow analysis
- Generate heuristic pseudo-C for ARM64 functions.
- Reconstruct control structures when possible, including `if` / `else`, loops, branches, and returns.
- CFG (*Control Flow Graph*) view with basic blocks and edges.
- Split view for Listing and pseudo-C side by side.
- Calls and cross-reference (XREF) views.
- Inspect callers and callees.

**Note:** pseudo-C is a heuristic reconstruction, not a perfect decompilation. Variable names, types, prototypes, and some control structures may be approximate. Pseudo-C support is currently limited to ARM64.

### Interface
- Android-oriented interface with a dark theme, red accents, and a light theme option.
- Main analysis views: Listing, Split, Pseudo-C, CFG, Calls, Hex, Functions, Strings, Symbols, ELF, Sections, Relocations, Search, and Bookmarks.
- Text-size adjustment.
- Bookmarks and navigation history.
- Symbol renaming and address comments.
- Copy pseudo-C and disassembly output.

### Local library and cache
- Store opened binaries in MiniIDA's local library.
- Reopen binaries from the library.
- Cache analysis results to speed up subsequent openings.
- Store display preferences locally.
- Store bookmarks, renames, and comments locally for the relevant binary.
- Export analysis information as JSON.

Library data and cache are stored locally in the WebView's browser storage (IndexedDB). Clearing app data or uninstalling the app may remove them. The current JSON export covers analysis metadata and related information; it is not a complete backup of the binary and the entire analysis session.

## Integrated MCP server

MiniIDA includes an MCP server that allows a compatible MCP client to query the binary currently loaded in the application.

### Available MCP tools

| Tool | Description |
|---|---|
| `get_binary_info` | General binary information, security properties, dependencies, and analysis state. |
| `get_sections` | ELF sections and program segments. |
| `get_memory_map` | Memory regions and permissions. |
| `get_functions` | Paginated function list, with name filtering. |
| `get_function` | Detailed information about a function. |
| `get_disassembly` | Disassembly starting at an address. |
| `get_xrefs` | Incoming and outgoing cross-references. |
| `get_strings` | Discovered strings and reference information. |
| `get_imports` | Imported symbols and PLT/GOT information. |
| `get_exports` | Exported symbols. |
| `get_call_graph` | Caller and callee graph. |
| `get_cfg` | Basic blocks and control-flow graph edges. |
| `get_pseudocode` | Heuristic ARM64 pseudo-C for a function. |
| `search_pattern` | Search bytes, text, immediate values, or names. |
| `get_annotations` | User renames, comments, and bookmarks. |
| `analyze` | Run full analysis if it has not already been completed. |
| `goto` | Navigate the MiniIDA interface to an address. |
| `set_name` | Rename a symbol when editing is enabled. |
| `set_comment` | Add or update a comment when editing is enabled. |

### Connecting to MCP

1. Open a binary in MiniIDA.
2. Run the analysis if needed.
3. Open the MCP menu and enable the server.
4. Choose local (`127.0.0.1`) or Wi-Fi network mode.
5. Copy the MCP configuration displayed by the app into your MCP client.

The server uses port `8765` by default and exposes the HTTP endpoint `POST /mcp`. Requests are authenticated with a Bearer token.

Example configuration — use the actual address and token displayed by the app:

```json
{
  "mcpServers": {
    "miniida": {
      "url": "http://127.0.0.1:8765/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_MINIIDA_TOKEN"
      }
    }
  }
}
```

`127.0.0.1` only works for a client running in the same network environment as the server. For a client on another device, use the network address displayed by MiniIDA and make sure both devices are connected to the same Wi-Fi network.

### MCP security
- The MCP server remains disabled until the user enables it.
- A Bearer token is required.
- Read-only mode is available.
- `set_name` and `set_comment` require editing to be explicitly enabled.
- In Wi-Fi mode, anyone with the server address and token can access the data exposed by the server. Do not share the token, and only enable the server on a trusted network.
- The server exposes the binary and analysis results currently loaded in MiniIDA.

## Android installation

1. Download the APK from the repository's **Releases** page.
2. Open the APK file on your Android device.
3. Allow installation from that source if Android prompts you.
4. Launch MiniIDA and open a file to analyze.

## Quick start

1. Open a `.so`, ELF, or raw binary file.
2. Let MiniIDA analyze the file.
3. Browse the ELF, Sections, Functions, Strings, Symbols, and Relocations views.
4. Select a function in the Listing.
5. Open **Pseudo-C** to inspect its heuristic reconstruction.
6. Open **CFG** to visualize basic blocks and branches.
7. Use **Calls** and XREFs to explore relationships between functions.
8. Add bookmarks, names, and comments to organize your analysis.
9. Enable MCP when an external client needs to query the current analysis.

## Known limitations
- The application interface is currently French-only. English localization is planned for a future update.
- Pseudo-C is heuristic and currently available for ARM64 only.
- Results depend on available symbols, binary quality, and detection heuristics.
- Heavily obfuscated binaries or binaries without useful metadata may produce approximate names and structures.
- Raw binary files do not provide all ELF metadata.
- MCP operates on the binary currently loaded in MiniIDA.
- The current JSON export is not a complete project backup.
- Full session backup in a project format and global export of pseudo-code for every function are planned improvements, not currently available features.

## Dependencies and licenses

MiniIDA includes components such as:
- [Capstone](https://github.com/capstone-engine/capstone)
- [Capstone.js](https://github.com/capstone-engine/capstone)

Review the licenses for bundled components and include all required notices before redistributing the application.

## Releases

Official APKs and release notes should be published through **GitHub Releases**.

For each release, it is recommended to provide:
- the APK file;
- the version number;
- a list of changes;
- tested architectures and features;
- the APK's SHA-256 checksum.

## Project status

MiniIDA is under active development. Features listed here describe the application build being documented. Planned features are identified separately so users can distinguish currently available functionality from future improvements.
