SuperGrafx Catalog, storage v4 (128 KiB blocks, 1 solid LZMA2 group of up to 32 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release of this platform.
- The RetroAchievements TurboGrafx-16 folder holds both platforms; it is split by extension (`.pce` here and `.sgx` in SuperGrafx, or the reverse). RA hashes drop a 512-byte copier header when size % 131072 = 512 (rcheevos).
- One schema for all twenty-three platforms: the header tables of the eight new platforms (Game Gear, PC Engine, SuperGrafx, MSX, MSX2, Virtual Boy, Game & Watch, Super A'Can: `pce_hardware`, `msx_hardware`, `vb_hardware`) and `rom_annotations` exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color, PC Engine<->SuperGrafx, MSX<->MSX2): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 14 ZIPs (nointro 10, retroachievements 4), 6.6 MiB (14 ROM files, 11.1 MiB uncompressed). Populated database: 5.5 MiB (83.4% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 7 ROM records, 6 games, 6 releases; DAT versions: 20250913-112105.
- RetroAchievements: 4 of 4 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 14.1 MiB/s (6 files); single file with a cold cache 0.084 s (ROM) / 0.122 s (TorrentZip) on average.
- Full audit of the populated database: 10 objects, 1 group, 14 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-SuperGrafx/blob/main/README.zh-CN.md)
