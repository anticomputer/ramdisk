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
./ramdisk create <size_mb> <name> [--encrypted] [--no-index]
```

Examples:
```bash
# Standard RAM disk
./ramdisk create 512 TempCache

# Encrypted RAM disk
./ramdisk create 512 SecureCache --encrypted

# RAM disk with indexing disabled (better performance)
./ramdisk create 512 FastCache --no-index

# Encrypted RAM disk with no indexing
./ramdisk create 512 SecureFastCache --encrypted --no-index
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

**Disable indexing (optional):**
- Add `--no-index` flag to prevent Spotlight, Time Machine, and other indexing services
- Improves performance by eliminating background indexing processes
- Prevents sensitive file names from being added to Spotlight's search index
- Useful for build directories, temporary workspaces, or performance-critical applications
- Can be combined with `--encrypted` (flags work in any order)
- Some operations may require sudo privileges (warnings shown if needed)

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
- Use HFS+ file system by default (APFS when using `--encrypted`)
- By default, Spotlight and other macOS services will index RAM disks unless `--no-index` is specified
- The `--no-index` flag may require sudo privileges for some operations (mdutil, tmutil)

## Safety and security features

- **RAM disk verification**: The `destroy` command verifies the device is actually a RAM disk before destroying it, preventing accidental destruction of regular disks
- **System RAM checks**: Prevents creating RAM disks larger than available system RAM
- **Force unmount protection**: If a RAM disk is in use, the script requires confirmation before force unmounting
- **Volume name validation**: Restricts volume names to safe characters to prevent injection attacks
- **Optional encryption**: APFS encryption with strong auto-generated passwords for sensitive data protection
- **Optional indexing control**: Disable Spotlight and Time Machine to improve performance and prevent metadata leakage

## Common use cases

**Standard RAM disks:**
- Build output directories for faster compilation
- Browser and application caches
- Working with large temporary files
- Reducing SSD wear for frequently written temporary data

**RAM disks with `--no-index`:**
- Software build directories (prevents Spotlight from indexing thousands of object files)
- Node.js `node_modules` or Python virtual environments
- Video editing scratch disks (prevents indexing large temporary render files)
- Database temporary directories
- Any high-performance workload where background processes interfere
- Privacy-conscious temporary storage (file names don't appear in Spotlight)

**Encrypted RAM disks:**
- Processing sensitive credentials or API keys
- Decrypting and working with confidential documents
- Security research or malware analysis
- Cryptocurrency operations
- Any temporary storage of data that must remain confidential

**Encrypted RAM disks with `--no-index`:**
- Maximum security: encryption + no traces in Spotlight index
- Ideal for handling highly sensitive data that requires both confidentiality and privacy
- Prevents both content access and metadata leakage

## Requirements

- macOS (uses `hdiutil` and `diskutil`)
- No special privileges required for basic operations
