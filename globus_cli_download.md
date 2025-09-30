# Globus CLI Data Transfer Guide

Quick reference for transferring sequencing data using Globus CLI.

## Prerequisites

```bash
# Install Globus CLI (one-time)
mamba create -n globus globus-cli
mamba activate globus

# Authenticate (do this once or when session expires)
globus login
```

## 1. Find Shared Data

When someone shares data with you via Globus:

```bash
# List collections shared with you
globus endpoint search --filter-scope shared-with-me

# View collection details
globus gcs collection show <collection-id>

# Browse the shared data
globus ls <collection-id>:/
```

**Save the collection ID** - you'll need it as your source.

## 2. Check Your Destination Endpoint

### Option A: Use institutional endpoint (preferred)

```bash
# Search for UCSC endpoints
globus endpoint search "ucsc"

# Test access
globus ls <endpoint-id>:/

# If accessible, save the endpoint ID
```

### Option B: Set up Globus Connect Personal

If institutional endpoints are non-functional:

```bash
# Download and extract (one-time setup)
cd ~
wget https://downloads.globus.org/globus-connect-personal/linux/stable/globusconnectpersonal-latest.tgz
tar xzf globusconnectpersonal-latest.tgz
cd globusconnectpersonal-*

# Setup (follow prompts to authenticate)
./globusconnectpersonal -setup

# Start in background with screen
screen -S globus
./globusconnectpersonal -start
# Press Ctrl+A, then D to detach

# Find your endpoint ID
globus endpoint search --filter-scope my-endpoints
```

**Save your endpoint ID** - you'll need it as your destination.

### Configure Accessible Paths

By default, Globus Connect Personal only allows access to your home directory. To access other paths:

```bash
# Stop the endpoint
cd ~/globusconnectpersonal-*
./globusconnectpersonal -stop

# Edit the config file
nano ~/.globusonline/lta/config-paths
```

Add paths you need access to (one per line):
```
/private/groups/russelllab/,0,1
```

Format: `path,writable(0=yes,1=no),shareable`

Save and restart:
```bash
./globusconnectpersonal -start
```

## 3. Transfer Data

```bash
globus transfer <source-collection-id>:/<source-path> \
                <destination-endpoint-id>:/<destination-path> \
                --recursive \
                --label "Description of transfer"
```

**Example:**
```bash
globus transfer 6fbe3d98-aaa6-482c-9815-a718be94d267:/ \
                ee0b4b13-9e29-11f0-ae47-0affca67c55f:/private/groups/russelllab/jodie/sequencing_data/scRNAseq/ \
                --recursive \
                --label "RNA-seq transfer from Duke"
```

The transfer runs server-side and continues even if you disconnect.

## 4. Monitor Transfer

```bash
# List all your transfers
globus task list

# Check specific transfer status
globus task show <task-id>

# View transfer events
globus task event-list <task-id> --limit 10

# Cancel a transfer if needed
globus task cancel <task-id>
```

You'll receive an email when the transfer completes.

## 5. Verify Transfer

```bash
# Check files arrived
ls -lh /path/to/destination/

# Verify checksums (if .md5 file provided)
cd /path/to/destination/
md5sum *.fastq.gz > computed.md5

# Compare with provided checksums
diff computed.md5 provided.md5
```

## Troubleshooting

**Endpoint is non-functional:**
- Try alternative institutional endpoints
- Set up Globus Connect Personal

**Permission denied:**
- Verify you have write access to destination path: `ls -ld /path/to/destination`
- Add the path to Globus Connect Personal config:
  ```bash
  cd ~/globusconnectpersonal-*
  ./globusconnectpersonal -stop
  nano ~/.globusonline/lta/config-paths
  # Add: /path/to/directory/,0,1
  ./globusconnectpersonal -start
  ```
- Check endpoint is active: `./globusconnectpersonal -status`

**Transfer stuck:**
- Check task status: `globus task show <task-id>`
- Ensure Globus Connect Personal is still running (if using it)
- Reattach to screen: `screen -r globus`

## Keeping Globus Connect Personal Running

If using your own endpoint, keep it running:

```bash
# Reattach to screen session
screen -r globus

# Check status
cd ~/globusconnectpersonal-*
./globusconnectpersonal -status

# Restart if needed
./globusconnectpersonal -start
```

### Restarting Globus Connect Personal

```bash
# Navigate to installation directory
cd ~/globusconnectpersonal-*

# Stop the endpoint
./globusconnectpersonal -stop

# Start it again
./globusconnectpersonal -start

# Or run in background
./globusconnectpersonal -start &

# Verify it's running
./globusconnectpersonal -status
```

**If running in screen session:**
```bash
# List screen sessions
screen -ls

# Reattach to globus session
screen -r globus

# Inside screen, restart
cd ~/globusconnectpersonal-*
./globusconnectpersonal -stop
./globusconnectpersonal -start

# Detach with Ctrl+A, then D
```

## Quick Reference

| Task | Command |
|------|---------|
| Find shared data | `globus endpoint search --filter-scope shared-with-me` |
| Browse collection | `globus ls <collection-id>:/` |
| Find your endpoints | `globus endpoint search --filter-scope my-endpoints` |
| Start transfer | `globus transfer <source>:<path> <dest>:<path> --recursive` |
| List transfers | `globus task list` |
| Check transfer | `globus task show <task-id>` |
