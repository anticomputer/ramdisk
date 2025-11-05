# ramdisk

A command-line tool for managing RAM disks on macOS.

## What it does

Creates, lists, and destroys RAM-based disks that are stored entirely in memory. RAM disks provide fast temporary storage for caches, build artifacts, or any data that doesn't need to persist across reboots.

## Installation

```bash
chmod +x ramdisk
# Optional: move to a directory in your PATH
sudo mv ramdisk /usr/local/bin/
```

## Usage

### Create a RAM disk

```bash
./ramdisk create <size_mb> <name>
```

Example:
```bash
./ramdisk create 512 TempCache
# Creates a 512MB RAM disk at /Volumes/TempCache
```

### List all RAM disks

```bash
./ramdisk list
```

Shows all active RAM disks with their device paths, mount points, and sizes.

### Destroy a RAM disk

```bash
./ramdisk destroy <name_or_device>
```

Examples:
```bash
./ramdisk destroy TempCache        # by volume name
./ramdisk destroy /dev/disk3       # by device path
```

### Help

```bash
./ramdisk help
```

## Important notes

- All data on a RAM disk is **permanently lost** when destroyed or when the system reboots
- RAM disks consume physical memory - creating a 1GB RAM disk uses 1GB of your system RAM
- RAM disks are mounted at `/Volumes/<name>`
- Use HFS+ file system by default

## Common use cases

- Build output directories for faster compilation
- Temporary caches that don't need to persist
- Working with large temporary files
- Reducing SSD wear for frequently written temporary data

## Requirements

- macOS (uses `hdiutil` and `diskutil`)
- No special privileges required for basic operations
