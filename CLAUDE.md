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

### Volume Name Validation (create command)
Uses whitelist approach for security:
- Only allows: `[a-zA-Z0-9 _-]` (letters, numbers, spaces, hyphens, underscores)
- Must start with letter or number
- Maximum 64 characters
- Prevents shell injection attacks from special characters like `$()`, backticks, etc.

### RAM Size Validation (create command)
- Checks total system RAM using `sysctl -n hw.memsize`
- Rejects if requested size > total RAM (prevents system crashes)
- Warns with confirmation if size > 80% of total RAM
- Additional warning for very large sizes (>32GB)

### Sector Calculation (create command)
macOS requires size in 512-byte sectors: `sectors = 2 * 1024 * size_mb`

### RAM Disk Detection (list command)
Parses `hdiutil info` output to find disks with `image-path: ram://...`. Must strip ANSI color codes from hdiutil output using `sed 's/\x1b\[[0-9;]*[a-zA-Z]//g'` for reliable pattern matching.

### Verification (destroy command)
Multi-layer safety checks before destroying:
1. Verifies device exists with `[[ -e "$device" ]]`
2. Verifies diskutil can access device
3. Uses `grep -w` (word boundary) to match device in hdiutil output, preventing false matches (e.g., /dev/disk1 vs /dev/disk10)
4. **Refuses entirely** if device is not a RAM disk (no bypass option)
5. If normal unmount fails, requires user confirmation before force unmount

## Security Considerations

### Why These Protections Matter

**Volume Name Validation**: Prevents shell injection attacks. Without proper validation, malicious input like `Test$(rm -rf /)` could execute arbitrary commands when passed to diskutil.

**RAM Size Checks**: Prevents system crashes. macOS will attempt to allocate the requested RAM even if it exceeds available memory, causing system instability or freeze.

**Device Verification**: Critical safety feature. Without strict RAM disk verification, a typo or bug could destroy a user's actual hard drive. The script uses word-boundary matching (`grep -w`) to avoid false positives and refuses to destroy non-RAM disks entirely.

**Force Unmount Confirmation**: Prevents data loss. If files are being written when force unmount occurs, data corruption can happen. The confirmation prompt gives users a chance to close applications properly.

### Design Philosophy

The script follows a "fail-safe" approach: when in doubt, refuse the operation rather than risk data loss or system instability. Error messages guide users to appropriate alternatives if they need to override safety checks.

## macOS-Specific Requirements

- Requires `hdiutil` (disk image utility)
- Requires `diskutil` (disk management utility)
- HFS+ file system used by default
- RAM disks mounted under `/Volumes/<name>`
- Uses `sysctl` for system RAM information
