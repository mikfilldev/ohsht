<div align="center">
  <h1><code>ohsht</code></h1>
  <p><em>oh sht, i forgot to backup</em></p>
  <p>A single-file Linux system backup tool. Zero dependencies.</p>
  <pre>sudo ohsht /dev/sda</pre>
  <br>
</div>

---

## What

`ohsht` backs up your entire Linux system (root, home, EFI, recovery — all physical mounts) to a target disk or partition using `rsync`. One file, standard library only.

## Usage

```bash
sudo ohsht <target> [options]
```

| Command | What it does |
|---|---|
| `sudo ohsht /dev/sda` | Partition, format, mount, rsync everything |
| `sudo ohsht /dev/sda1` | Mount existing partition, rsync everything |
| `sudo ohsht /dev/sda1 --fmt` | Format partition first, then backup |
| `sudo ohsht /dev/sda1 -R` | Restore — reverse the rsync |
| `sudo ohsht /dev/sda -n` | Dry-run — preview without touching anything |

### Examples

```bash
# Full backup to a new disk
sudo ohsht /dev/sda

# Backup to an existing partition
sudo ohsht /dev/sdb2

# Restore from backup
sudo ohsht /dev/sda1 -R

# See what would happen
sudo ohsht /dev/sda -n
```

## How it works

1. **Detects** your physical mounts from `/proc/mounts` (root, boot, home, recovery, etc.)
2. **Partitions** the target disk (if a whole disk is given) with `parted`
3. **Formats** with `ext4` (label: `OHSHT`)
4. **Mounts** at `/mnt/ohsht`
5. **Rsyncs** each mount point separately using:
   ```
   rsync -aAXHSx --numeric-ids --info=progress2
   ```
6. **Writes** metadata: `BACKUP_INFO.txt` and `RSYNC_EXCLUDES.txt`
7. **Unmounts** when done

### Excluded by default

System pseudo-fs (`/dev`, `/proc`, `/sys`, `/tmp`, `/run`, `/mnt`, `/media`), swap files, apt cache, `.cache` dirs.

### Backup structure

```
/mnt/ohsht/
├── root/           ← /
├── home/           ← /home
├── boot-efi/       ← /boot/efi
├── recovery/       ← /recovery
├── BACKUP_INFO.txt
└── RSYNC_EXCLUDES.txt
```

## Install

```bash
# Clone
git clone git@github.com:mikfilldev/ohsht.git
cd ohsht

# Optional: put in PATH
sudo ln -s "$PWD/ohsht" /usr/local/bin/ohsht
```

Requires: `python3`, `rsync`, `parted`, `mkfs.ext4`, `mount`.

## Why

- One file. 204 lines. No `pip install` anything.
- Standard library only: `argparse`, `subprocess`, `os`, `sys`, `time`, `pathlib`
- Calls system tools (`rsync`, `parted`, `mount`) via `subprocess` — nothing to install
- Dry-run to preview before committing

## Restore

Boot from a live USB, mount your backup, then:

```bash
sudo ohsht /dev/sda1 -R
```

This reverse-rsyncs each backup directory back to its original mount point.

## License

MIT
