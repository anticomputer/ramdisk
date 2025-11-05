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

**Volume name requirements:**
- Must start with a letter or number
- Can only contain letters, numbers, spaces, hyphens, and underscores
- Maximum 64 characters
- Cannot contain special characters like `$`, `(`, `)`, `/`, `:`, etc.

**Size validation:**
- The script checks available system RAM and prevents creation if size exceeds total RAM
- Warns with confirmation prompt if size exceeds 80% of total RAM

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

## Safety features

- **RAM disk verification**: The `destroy` command verifies the device is actually a RAM disk before destroying it, preventing accidental destruction of regular disks
- **System RAM checks**: Prevents creating RAM disks larger than available system RAM
- **Force unmount protection**: If a RAM disk is in use, the script requires confirmation before force unmounting
- **Volume name validation**: Restricts volume names to safe characters to prevent injection attacks

## Common use cases

- Build output directories for faster compilation
- Temporary caches that don't need to persist
- Working with large temporary files
- Reducing SSD wear for frequently written temporary data

## Requirements

- macOS (uses `hdiutil` and `diskutil`)
- No special privileges required for basic operations
