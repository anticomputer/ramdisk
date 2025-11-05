# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file bash script (`ramdisk`) for managing RAM disks on macOS. Uses `hdiutil` for creating RAM-backed disk images and `diskutil` for formatting/mounting.

## Testing

Test the script manually:

```bash
# Create a test RAM disk
./ramdisk create 50 TestDisk

# Verify it appears in the list
./ramdisk list

# Verify it's accessible
ls /Volumes/TestDisk

# Clean up
./ramdisk destroy TestDisk
```

## Key Implementation Details

### RAM Disk Detection (list command)
Parses `hdiutil info` output to find disks with `image-path: ram://...`. Must strip ANSI color codes from hdiutil output using `sed 's/\x1b\[[0-9;]*[a-zA-Z]//g'` for reliable pattern matching.

### Sector Calculation (create command)
macOS requires size in 512-byte sectors: `sectors = 2 * 1024 * size_mb`

### Verification (destroy command)
Before destroying, verifies the device exists in hdiutil info to prevent accidental deletion of non-RAM disks. Includes confirmation prompt if verification fails.

## macOS-Specific Requirements

- Requires `hdiutil` (disk image utility)
- Requires `diskutil` (disk management utility)
- HFS+ file system used by default
- RAM disks mounted under `/Volumes/<name>`
