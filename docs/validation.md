# Documentation validation

The README descriptions were checked against the current C sources and disposable execution fixtures on 2026-10-11. The original Word reports were used to identify the design diagrams and compare their intended behavior with the implementation. Reports and extracted screenshots were not added to Git.

## Environment

- WSL2, x86_64, kernel `6.18.33.2-microsoft-standard-WSL2`.
- Intel Core Ultra 5 125H; approximately 7.5 GiB total Linux VM memory.
- Source and temporary fixtures on the VM's ext4 filesystem.
- GCC 9.5 for the organizer and inspector; GCC 13.3 for the daemon.
- Make, Python 3, and e2fsprogs for fixture creation.

The daemon's ordinary GCC 9 build failed on declarations directly after `case` labels. A fresh build with `make CC=gcc-13` succeeded. Organizer and inspector built with ordinary `make`. No program logic was changed during this documentation work.

## Execution checks

| Tool | Fixture and checks | Observed result |
| --- | --- | --- |
| File tree organizer | Nested files with txt, c, and no extension; hidden file; excluded subtree; size/permission view | Three eligible files copied with matching contents; originals retained; hidden/excluded files omitted |
| File tree organizer | Recent and two-hour-old txt files, with `-t 3600` | Recent file admitted; old file omitted |
| File tree organizer | Two nested sources with the same basename, using the interactive collision prompt | `select 2` copied the second source's verified contents |
| File tree organizer | Five full requests, each with 512 files and 128 MiB | 2,560 copies hash-verified; median wall time 0.122 s; [measurement details](https://github.com/jaeunda/file-tree-organizer/blob/main/docs/measurements.md) |
| File organizer worker | Duplicate pair with distinct mtimes, tested in all three modes | Newer selected in mode 1; older in mode 2; pair omitted in mode 3 |
| File organizer worker | A source addition and deletion between scans | New destination copied; stale destination removed |
| File organizer daemon | Launch in isolated HOME, exit shell, create another source, revise extension filter, remove registration | Daemon continued, copied the later file, applied the filter, and removed its registration |
| ext2 inspector | Fresh 32 MiB images, 1 KiB and 4 KiB blocks; recursive tree, modes/sizes, short and limited text, empty file, missing path | Expected output and missing-path error reproduced on both |
| ext2 inspector | 480,000-byte text file, 30,000 lines aligned to 16-byte boundaries | All lines emitted on both images; 1 KiB fixture exercised double-indirect traversal |
| ext2 inspector | Hash before and after commands | Image SHA-256 unchanged on both fixtures |

The validation scripts used temporary HOME and fixture directories so the daemon checks did not change existing registration storage. The test daemon was explicitly terminated after its check.

## Reproduced defects and source-derived limits

An initial inspector fixture used 17-byte lines. Lines crossing block boundaries were split, so the output did not preserve complete source lines. Separate requests reproduced removal of blank lines and a longer pathname component incorrectly matching the prefix `hello.txt`.

Source review also identified the organizer's missing copy-success return and misplaced external-tool waits; the daemon's unchanged running sleep interval, uninstalled signal handler, mode-3 behavior with a third duplicate, shorter config rewrites, and separate lock scopes; and the inspector's direct-only directory traversal, text formatting, and incomplete image/error validation. The detailed READMEs describe those boundaries. Passing fixtures are not presented as proof of complete correctness.

## Diagram coverage

| Development report | Redesigned SVGs included in the project README |
| --- | --- |
| P1: shell, tree worker, extension/file lists, arrange worker | `command-dispatch.svg`, `tree-traversal.svg`, `extension-index.svg`, `arrange-pipeline.svg` |
| P2: manager commands, daemon loop, source/output lists | `daemon-control.svg`, `daemon-cycle.svg`, `directory-merge.svg` |
| P3: prompt dispatch, tree flow, print flow | `command-shell.svg`, `tree-pipeline.svg`, `print-pipeline.svg` |

The collection overview, three compact project diagrams in the main README, and ext2 inode-addressing diagram are additional figures. All fifteen figures are SVG vectors with outlined Noto Sans labels, descriptive titles, and a shared restrained palette. Font licenses are included with the assets. The redraws use actual source names and behavior where the original diagram or prose differed from the code.
