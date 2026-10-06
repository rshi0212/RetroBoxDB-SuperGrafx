# RetroBoxDB SuperGrafx

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for NEC PC Engine SuperGrafx. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 14 source ZIPs, 6.6 MiB (No-Intro 10, RetroAchievements sets 4); 14 ROM files, 11.1 MiB uncompressed |
| Stored size | populated database 5.5 MiB; public Catalog 1.9 MiB (no ROM data) |
| Ratio | 83.4% of the source ZIPs, 49.4% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 128 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 32 MiB (32 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (6 files, each checked against the DAT hashes): 14.1 MiB/s, 49 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.084 s, TorrentZip 0.122 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.SuperGrafx.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-SuperGrafx/releases/latest/download/RetroBoxDB.SuperGrafx.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-supergrafx-games.csv) / [summary](reports/ra-supergrafx.json), [build report](reports/supergrafx-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

6 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-supergrafx.json`): smallest 512 KiB / 32 MiB at 2.20 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 128 KiB / 32 MiB at 2.20 MiB. ZIPs 6.59 MiB, per-file LZMA 2.77 MiB.

- HuCards have no internal header. `pce_hardware` records image facts only: a 512-byte copier header (file size % 8192 = 512; cut into its own block so the body deduplicates with headerless dumps), ROM size, 8 KiB banks, bytes of a partial bank and the reset vector at the end of the first bank. Mapper and region need external evidence and are not guessed.
- RetroAchievements lists PC Engine and SuperGrafx games under one console (8) and one folder (TurboGrafx-16). Both databases import that folder; this one keeps `.sgx` files and skips `.pce` files, which [RetroBoxDB-PCEngine](https://github.com/rshi0212/RetroBoxDB-PCEngine) holds. RA hashes drop a 512-byte copier header when the size % 131072 = 512 (rcheevos). The RA report covers only games tied to this database.
- The platform is small (5 SuperGrafx games in the DAT, 7 ROMs with the aftermarket and RetroAchievements files), so the populated database is about as large as its ZIPs: the embedded engine, documents and reports take about 3 MiB in every database.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 7 / 6 / 6 |
| DAT coverage per version | 20250913-112105: 6/6 |
| Local ROMs in no DAT | 1 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 3, RA only 1, hash not in the latest RA snapshot 0 ([list](reports/ra-supergrafx-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-supergrafx-missing.csv) |
| No-Intro DB Export + Dump Log 20250913-112105 | 7 archives, 7 file identities, 0 documented hardware assertions; Dump Log Verified 0 |
| RetroAchievements (console 8) | 4 games with achievements: 4 with a local ROM (4 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 0 without a No-Intro counterpart |
| Chinese names | 5 of 5 rows translated (5 unique); 5 local ROMs have a Chinese name |
| Populated-database audit | 10 objects, 1 groups, 14 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.SuperGrafx.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.SuperGrafx.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.SuperGrafx.sqlite --discover --ra --catalog RetroBoxDB.SuperGrafx.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
