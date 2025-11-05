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
./ramdisk create <size_mb> <name> [--encrypted]
```

Examples:
```bash
# Standard RAM disk
./ramdisk create 512 TempCache

# Encrypted RAM disk
./ramdisk create 512 SecureCache --encrypted
```

**Volume name requirements:**
- Must start with a letter or number
- Can only contain letters, numbers, spaces, hyphens, and underscores
- Maximum 64 characters
- Cannot contain special characters like `$`, `(`, `)`, `/`, `:`, etc.

**Size validation:**
- The script checks available system RAM and prevents creation if size exceeds total RAM
- Warns with confirmation prompt if size exceeds 80% of total RAM

**Encryption (optional):**
- Add `--encrypted` flag to create an encrypted APFS volume
- A strong 24-character random password is automatically generated
- **IMPORTANT**: Save the password immediately - it cannot be recovered
- Use for sensitive data like credentials, API keys, or personal information

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

## Safety and security features

- **RAM disk verification**: The `destroy` command verifies the device is actually a RAM disk before destroying it, preventing accidental destruction of regular disks
- **System RAM checks**: Prevents creating RAM disks larger than available system RAM
- **Force unmount protection**: If a RAM disk is in use, the script requires confirmation before force unmounting
- **Volume name validation**: Restricts volume names to safe characters to prevent injection attacks
- **Optional encryption**: APFS encryption with strong auto-generated passwords for sensitive data protection

## Common use cases

**Standard RAM disks:**
- Build output directories for faster compilation
- Browser and application caches
- Working with large temporary files
- Reducing SSD wear for frequently written temporary data

**Encrypted RAM disks:**
- Processing sensitive credentials or API keys
- Decrypting and working with confidential documents
- Security research or malware analysis
- Cryptocurrency operations
- Any temporary storage of data that must remain confidential

## Requirements

- macOS (uses `hdiutil` and `diskutil`)
- No special privileges required for basic operations
