# Installation and Usage

version 0.6.0

## Quick Start

### 1. Install Dependencies

```bash
sudo apt-get update
sudo apt-get install python3 python3-aiohttp xdelta3
```

**Note**: `python3-aiohttp` is available in:
- Debian Testing (Bookworm+) and Unstable
- Ubuntu 24.04 (Noble) and later

For older Ubuntu versions, you may need to use backports or compile from source.

### 2. Install apt-mirror.py

```bash
# Copy the script to a system location
sudo cp apt-mirror.py /usr/local/bin/apt-mirror
sudo chmod +x /usr/local/bin/apt-mirror
```

Or if replacing the Perl version:
```bash
sudo cp apt-mirror.py /usr/bin/apt-mirror
sudo chmod +x /usr/bin/apt-mirror
```

### 3. Configure

Edit `/etc/apt/mirror.list` (or your custom config file):

```bash
sudo nano /etc/apt/mirror.list
```

Example configuration:
```
set base_path         /var/spool/apt-mirror
set mirror_path       $base_path/mirror
set skel_path         $base_path/skel
set var_path          $base_path/var
set defaultarch       amd64
set nthreads          20
set limit_rate        100m
set enable_diffs      1
set diff_algorithm    xdelta3

deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
```

### 4. Run

```bash
sudo apt-mirror /etc/apt/mirror.list
```

Or with the default config:
```bash
sudo apt-mirror
```

## New Configuration Options

Add these to your `mirror.list` for new features:

```
# Enable diff generation
set enable_diffs 1

# Choose diff algorithm (xdelta3, bsdiff, or rsync)
set diff_algorithm xdelta3

# Where to store diffs
set diff_storage_path $base_path/diffs

# Only create diffs if they're smaller than 50% of original
set max_diff_size_ratio 0.5

# Retry configuration
set retry_attempts 5
set retry_delay 2.0

# Verify checksums after download
set verify_checksums 1

# Resume partial downloads
set resume_partial_downloads 1
```

## Verification

Check that dependencies are installed:

```bash
python3 -c "import aiohttp; print('✓ aiohttp OK')"
which xdelta3 && echo "✓ xdelta3 OK"
```

## Troubleshooting

### python3-aiohttp not found

If `python3-aiohttp` is not available in your repository:

1. **Check if it's in backports:**
   ```bash
   sudo apt-get install -t <release>-backports python3-aiohttp
   ```

2. **For very old systems**, you can modify the script to use `urllib` instead (less efficient but works)

### Permission Errors

Make sure you run as a user with write access to the mirror directory:
```bash
sudo apt-mirror
```

Or configure proper permissions:
```bash
sudo chown -R apt-mirror:apt-mirror /var/spool/apt-mirror
```

## Performance Tips

1. **Adjust nthreads** based on your bandwidth and CPU:
   - For fast connections: 20-50 threads
   - For slower connections: 5-10 threads

2. **Enable diffs** to save bandwidth on updates:
   ```
   set enable_diffs 1
   ```

3. **Use limit_rate** to avoid saturating your connection:
   ```
   set limit_rate 100m
   ```

## Compatibility

- **Backward compatible** with existing `mirror.list` files
- **Same directory structure** as Perl version
- **Drop-in replacement** - can replace Perl script directly

