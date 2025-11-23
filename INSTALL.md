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

```bash
set base_path         /var/spool/apt-mirror
set mirror_path       $base_path/mirror
set skel_path         $base_path/skel
set var_path          $base_path/var
set postmirror_script $var_path/postmirror.sh
set defaultarch       amd64
set run_postmirror    0
set nthreads          20
set limit_rate        100m
set _tilde            0
set unlink            1
set use_proxy         off
set http_proxy        127.0.0.1:3128
set proxy_user        user
set proxy_password    password
set enable_diffs      1
set diff_algorithm    xdelta3
set diff_storage_path $base_path/diffs
set max_diff_size_ratio 0.5
set retry_attempts    5
set retry_delay       2.0
set verify_checksums  1
set resume_partial_downloads 1

# Ubuntu 24.04 LTS (Noble Numbat) repositories
deb http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu noble-security main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu noble-backports main restricted universe multiverse

# Source repositories (optional, uncomment if needed)
#deb-src http://archive.ubuntu.com/ubuntu noble main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu noble-updates main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu noble-security main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu noble-backports main restricted universe multiverse

clean http://archive.ubuntu.com/ubuntu
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

```bash
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

# Proxy configuration (for authenticated proxies)
set use_proxy on
set http_proxy http://proxy.example.com:3128
set https_proxy http://proxy.example.com:3128
set proxy_user your_proxy_username
set proxy_password your_proxy_password

# Unlink option: For hardlinked directories support. When enabled, unlinks destination files before copying if they differ. This is necessary when using hardlinks - you cannot overwrite a hardlinked file directly, you must unlink it first. (Note: wget --unlink is no longer used since we use async downloads, but this option still applies to file copying operations)
set unlink 1
```

## Verification

Check that dependencies are installed:

```bash
python3 -c "import aiohttp; print('✓ aiohttp OK')"
which xdelta3 && echo "✓ xdelta3 OK"
```

## Web Server

apt-mirror is only half the equation. While apt-mirror creates the files framework for serving a mirror, you will still need a web server to serve the files.

We recommend either Nginx or Apache. There are a lot of elements you can configure for web servers, so here are some example configurations that might work for you.
These are only examples, you will need to sort out your own configuration if it is more complex.

### Nginx Configuration

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;

    # Point the web root directly to the mirror directory
    root /var/spool/apt-mirror/mirror;

    location /ubuntu/ {
        autoindex on;
        alias /var/spool/apt-mirror/mirror/archive.ubuntu.com/ubuntu/;
    }
}
```

### Apache2 Configuration

```apache
<VirtualHost *:80>
    ServerName _
    
    # Point the web root directly to the mirror directory
    DocumentRoot /var/spool/apt-mirror/mirror
    
    <Directory /var/spool/apt-mirror/mirror>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    
    # Alias for Ubuntu repository
    Alias /ubuntu /var/spool/apt-mirror/mirror/archive.ubuntu.com/ubuntu
    
    <Directory /var/spool/apt-mirror/mirror/archive.ubuntu.com/ubuntu>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
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

   ```bash
   set enable_diffs 1
   ```

3. **Use limit_rate** to avoid saturating your connection:

   ```bash
   set limit_rate 100m
   ```

## Compatibility

- **Backward compatible** with existing `mirror.list` files
- **Same directory structure** as Perl version
- **Drop-in replacement** - can replace Perl script directly
