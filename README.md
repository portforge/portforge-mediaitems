# PortForge MediaItems

The official MediaItems library for [PortForge](https://github.com/portforge/portforge-app) — a desktop application for managing and launching game ports.

---

## How it works

PortForge reads this library at runtime. It never writes to it — all user data (installed games, ROM files, play history) lives in a separate writable folder chosen by the user at first launch.

---

## Repository structure

```
MediaItems/
├── VideoGame/
│   └── <Title> · <Year>/
│       ├── .mediaitem.json     - game metadata (title, description, artwork list, …)
│       └── .artwork/           - cover art, banners, box scans, …
│
├── VideoGameVersion/
│   └── <Title> · <Year>/
│       ├── .mediaitem.json     - version metadata (platform, ROM dependencies, …)
│       ├── .install.json       - install/build spec for PortForge
│       └── .artwork/           - version-specific artwork (optional)
│
└── VideoGameRom/
    └── <Title> · <Platform>/
        └── .mediaitem.json     - ROM metadata (checksums, file formats, No-Intro ID, …)
```

Each folder name doubles as the item's canonical identifier (`_itemTitle`). The `·` separator between title and year/platform is part of the naming convention and must be preserved exactly.

---

## Item types

### `VideoGame`

Top-level record for a game title. Contains a description, artwork references, and a `versions` list linking to known `VideoGameVersion` entries.

### `VideoGameVersion`

A specific release or port of a game — the original console release, a PC port, a fan remake, etc. This is what PortForge displays in the library and what users install.

Key fields in `.mediaitem.json`:

| Field             | Description                                                     |
| ----------------- | --------------------------------------------------------------- |
| `versionType`     | `"Original"`, `"FanPort"`, `"Remake"`, …                        |
| `platforms`       | Platforms this version runs on: `"Linux"`, `"Windows"`, `"Mac"` |
| `romDependencies` | ROMs required to install this version (matched by checksum)     |
| `artwork`         | List of artwork files present in `.artwork/`                    |

If a version requires compilation from source, an `.install.json` file defines the build process. See [Install specs](#install-specs) below.

### `VideoGameRom`

Metadata for a specific ROM dump, including checksums for every supported file format. PortForge matches files in the user's ROM library against these checksums to determine which ROMs are available.

Key fields:

| Field       | Description                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------- |
| `platform`  | Console abbreviation, e.g. `"N64"`, `"NES"`                                                                         |
| `noIntroId` | No-Intro database identifier(s)                                                                                     |
| `formats`   | Array of known file variants, each with `filename`, `filesize`, `ext`, and `checksums` (MD5, SHA-1, SHA-256, CRC32) |

---

## Install specs

When a `VideoGameVersion` requires a build step, a `.install.json` file is placed alongside `.mediaitem.json`. The file contains an array of specs — PortForge picks the first one whose `targetPlatforms` matches the current OS.

Each spec can declare:

- **`dependencies`** — command-line tools that must be present (e.g. `make`, `gcc`). PortForge checks these before starting and reports missing ones clearly.
- **`args`** — user-configurable choices shown in the UI before install (e.g. ROM region, texture pack). Options that require a ROM file are automatically marked available or unavailable based on the user's library.
- **`steps`** — the ordered build process: `fetch`, `extract`, `make`, `copy`, `move`, `createDir`, `touch`, `deletePath`, `defineExecutable`.
- **`uninstallSteps`** — optional steps run when the user uninstalls. If absent, PortForge removes the `install/` directory.

For the full step reference and security model, see [`docs/build-system.md`](https://github.com/portforge/portforge-app/blob/main/docs/build-system.md) in the main PortForge repository.

---

## Contributing

Contributions of new game entries, corrected checksums, improved descriptions, and better artwork are welcome.

### Adding a new game port

1. Create a `VideoGameVersion/<Title> · <Year>/` folder.
2. Write `.mediaitem.json` — fill in `_itemTitle`, `title`, `versionType`, `platforms`, and any `romDependencies`.
3. If the game must be compiled, add `.install.json` following the build system spec.
4. Add cover art to `.artwork/` (PNG preferred, named `Cover · en · 1.png`).
5. If a parent `VideoGame` entry doesn't exist yet, create one under `VideoGame/`.
6. If new ROMs are required, add entries under `VideoGameRom/` with full checksums for all known file variants.

### Checksums

ROM checksums must be sourced from the [No-Intro](https://no-intro.org) or the [Redump](https://redump.om) database where possible. Include MD5, SHA-1, SHA-256, and CRC32. Multiple format variants (e.g. Big Endian `.z64`, Byte Swapped `.v64`) should each be listed as a separate entry in the `formats` array.

### Artwork

- Artwork files go in the `.artwork/` subdirectory of the relevant item folder.
- Use the naming pattern `<ArtworkType> · <language> · <index>.<ext>` for localised artwork (e.g. `Cover · en · 1.png`) or `<ArtworkType> · <index>.<ext>` for language-neutral artwork (e.g. `Banner · 1.png`).
- PNG is preferred. JPEG is accepted for scanned box art.

---

## Credits

- [No-Intro](https://no-intro.org/) and [Redump](http://redump.org/) for cataloging original game media
- The contributors to [SteamGridDB](https://www.steamgriddb.com/) for game artwork

---

## Licence

- All game specific artwork belongs to the respective rights holders