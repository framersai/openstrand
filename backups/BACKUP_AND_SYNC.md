# Backup & Sync

OpenStrand provides powerful backup and synchronization features that mirror your notes and assets to local or cloud storage, with optional Git integration for version control.

## Overview

The backup system creates a **one-way mirror** from your database to the filesystem, ensuring your content is always accessible outside the app. This enables:

- **AI Tool Compatibility**: Works seamlessly with Claude Code, OpenAI Codex, and other AI assistants
- **Offline Access**: Access your notes even when the database is unavailable
- **Version Control**: Optional Git integration tracks every change
- **Data Portability**: Export your entire knowledge base as standard files
- **Disaster Recovery**: Automatic backups protect against data loss

## Directory Structure

When mirroring is enabled, OpenStrand creates the following structure:

```
/your/content/root/
├── notes/              # Your notes as JSON or Markdown
│   ├── 2025-01-15/
│   │   ├── my-note-abc123.json
│   │   └── another-note-def456.md
│   └── 2025-01-16/
│       └── daily-log-ghi789.json
├── assets/             # Media files organized by strand
│   ├── strands/
│   │   ├── abc123/
│   │   │   ├── photo.jpg
│   │   │   └── recording.webm
│   │   └── def456/
│   │       └── diagram.png
│   └── shared/         # Deduplicated shared assets
│       └── logo-hash.png
├── .openstrand/        # Metadata and configuration
│   ├── index.json      # Content catalog
│   ├── config.json     # Mirror settings
│   └── logs/           # Sync operation logs
└── .git/               # Git repository (if enabled)
    ├── objects/
    ├── refs/
    └── config
```

## Configuration

### Community Edition

Community users have full control over their storage settings:

1. Navigate to **Settings → Storage**
2. Set your **Content Root Path** (e.g., `/Users/you/Documents/OpenStrand`)
3. Enable **Local Mirroring**
4. Optionally enable **Git Integration**
5. Configure advanced options:
   - **Delete Behavior**: confirm, trash, or immediate
   - **Max Asset Size**: 512 MB default
   - **Compression**: lossless (recommended), lossy, or none
   - **Deduplication**: enabled by default to save space

### Teams Edition

Team administrators can set organization-wide defaults:

1. Navigate to **Teams → Storage** (admin only)
2. Configure **Default Team Policy**
3. Choose storage provider: **Local** or **S3-Compatible**
4. Set **Override Controls**:
   - **Allow member overrides**: members can customize settings
   - **Allow opt-out**: members can disable mirroring
5. Click **Enforce Now** to apply policy to all members immediately

## Storage Providers

### Local Filesystem

**Best for**: Personal use, desktop apps, self-hosted deployments

- Stores files directly on your computer or server
- Zero cloud dependencies
- Works offline
- Fast read/write performance
- Compatible with file sync tools (Dropbox, iCloud, Syncthing)

**Configuration**:
```json
{
  "provider": "local",
  "contentRootPath": "/Users/you/Documents/OpenStrand",
  "mirrorMode": "mirror"
}
```

### S3-Compatible Cloud Storage

**Best for**: Teams, cloud deployments, distributed access

Supports:
- **AWS S3**
- **Linode Object Storage**
- **DigitalOcean Spaces**
- **MinIO** (self-hosted)
- **Backblaze B2**

**Configuration**:
```json
{
  "provider": "s3",
  "bucket": "openstrand-team-backups",
  "prefix": "team-data/",
  "mirrorMode": "mirror"
}
```

**Environment Variables** (for S3):
```bash
S3_BUCKET=your-bucket-name
S3_REGION=us-east-1
S3_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
S3_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
S3_ENDPOINT=https://us-east-1.linodeobjects.com  # For Linode
```

## Git Integration

### Local Git Repository

When Git is enabled, OpenStrand automatically:

1. **Initializes** a Git repository in your content root
2. **Commits** changes every ~60 seconds (batched)
3. **Generates** meaningful commit messages
4. **Creates** a `.gitignore` to exclude temporary files

**Example commit messages**:
```
mirror: update 3 notes, 1 asset

- notes/2025-01-15/my-note-abc123.json
- notes/2025-01-15/another-note-def456.md
- assets/strands/abc123/photo.jpg
```

### Git Workflow

**View history**:
```bash
cd /your/content/root
git log --oneline --graph
```

**Restore a previous version**:
```bash
git checkout HEAD~5 notes/2025-01-15/my-note-abc123.json
```

**Create a branch for experiments**:
```bash
git checkout -b experiment
# Make changes...
git checkout main  # Return to main branch
```

**Remote Git Push** (optional, see [Remote Git](#remote-git-push)):
```bash
git remote add origin https://github.com/you/openstrand-backup.git
git push -u origin main
```

## CLI & AI Tool Compatibility

### Claude Code

OpenStrand's directory structure is optimized for Claude Code:

```bash
# Index your notes
claude index /your/content/root/notes

# Ask questions about your knowledge base
claude ask "What are my notes about machine learning?"

# Generate summaries
claude summarize /your/content/root/notes/2025-01-15/
```

### OpenAI Codex

Use Codex to search and analyze your notes:

```python
import openai

# Read your notes
with open('/your/content/root/notes/2025-01-15/my-note.json') as f:
    note = json.load(f)

# Ask Codex to summarize
response = openai.Completion.create(
    model="code-davinci-002",
    prompt=f"Summarize this note: {note['content']}"
)
```

### Custom Scripts

**Find all notes with a specific tag**:
```bash
cd /your/content/root/notes
grep -r '"tags".*"machine-learning"' . | cut -d: -f1
```

**Count notes by month**:
```bash
ls -1 notes/ | cut -d- -f1-2 | uniq -c
```

**Export to Markdown**:
```bash
for file in notes/**/*.json; do
  jq -r '.content.text' "$file" > "${file%.json}.md"
done
```

## Sync Operations

### Manual Sync

Trigger a full sync from the UI:

1. Navigate to **Settings → Storage**
2. Click **Run Sync Now**
3. Monitor progress in the status panel

Or via API:
```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:8000/api/v1/storage/sync
```

### Automatic Sync

Mirroring happens automatically when:
- A note is created or updated
- An asset is uploaded
- A strand is deleted (respects prune behavior)

### Sync Status

Check sync status via API:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:8000/api/v1/storage/status
```

Response:
```json
{
  "policy": {
    "provider": "local",
    "contentRootPath": "/Users/you/Documents/OpenStrand",
    "mirrorMode": "mirror",
    "gitEnabled": true
  },
  "gitStatus": {
    "initialized": true,
    "lastCommit": "2025-01-15T10:30:00Z",
    "commitCount": 42
  },
  "stats": {
    "notesCount": 127,
    "assetsCount": 45,
    "totalSizeMB": 234.5
  }
}
```

## Prune Behavior

When you delete content in OpenStrand, the prune behavior determines what happens to mirrored files:

### Confirm (Default)

- Shows a confirmation dialog before deleting
- Prevents accidental data loss
- **Recommended for most users**

### Trash

- Moves deleted files to `.trash/` folder
- Preserves files for 30 days (configurable)
- Allows easy recovery
- **Recommended for teams**

### Immediate

- Deletes files immediately without confirmation
- Frees up storage space instantly
- **Use with caution**

## Advanced Features

### Deduplication

OpenStrand automatically deduplicates assets by content hash:

- Identical files are stored only once
- Saves storage space (up to 70% for media-heavy vaults)
- Transparent to users
- Works across all strands

**Example**:
```
assets/
├── strands/
│   ├── abc123/
│   │   └── logo.png -> ../../shared/logo-sha256.png
│   └── def456/
│       └── logo.png -> ../../shared/logo-sha256.png
└── shared/
    └── logo-sha256.png  # Actual file stored once
```

### Compression

**Lossless** (default):
- No quality loss
- Smaller file sizes (10-30% reduction)
- Recommended for text, code, diagrams

**Lossy**:
- Aggressive compression
- Smaller file sizes (50-80% reduction)
- May reduce quality
- Recommended for large media libraries

**None**:
- No compression
- Original file sizes
- Fastest performance

### Two-Way Sync (Planned)

Future versions will support bidirectional sync:

- Edit files directly in your file manager
- Changes automatically sync back to OpenStrand
- Conflict resolution with merge strategies
- **Status**: Under development (see [Roadmap](./ROADMAP.md))

## Troubleshooting

### Mirror Not Working

**Check policy**:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:8000/api/v1/storage/policy
```

**Verify content root exists**:
```bash
ls -la /your/content/root
```

**Check permissions**:
```bash
# macOS/Linux
chmod 755 /your/content/root

# Windows (PowerShell)
icacls "C:\your\content\root" /grant Users:F
```

### Git Not Initializing

**Manual initialization**:
```bash
cd /your/content/root
git init
git add .
git commit -m "Initial commit"
```

**Check Git installation**:
```bash
git --version
```

### S3 Connection Errors

**Test credentials**:
```bash
aws s3 ls s3://your-bucket-name --profile openstrand
```

**Verify endpoint**:
```bash
curl -I https://us-east-1.linodeobjects.com
```

### Large Sync Times

**Reduce asset size**:
- Enable compression
- Lower max asset size
- Use lossy compression for media

**Exclude large files**:
Add to `.openstrand/config.json`:
```json
{
  "excludePatterns": [
    "*.mp4",
    "*.mov",
    "assets/videos/**"
  ]
}
```

## Security & Privacy

### Local Storage

- Files stored unencrypted by default
- Use full-disk encryption (FileVault, BitLocker) for protection
- Restrict file permissions to your user account

### S3 Storage

- Enable server-side encryption (SSE-S3 or SSE-KMS)
- Use IAM roles with least-privilege access
- Enable bucket versioning for recovery
- Configure lifecycle policies for cost optimization

### Git Commits

- Commits are local by default (no remote push)
- Sensitive data is **not** automatically redacted
- Review `.gitignore` before pushing to remote

## Remote Git Push

**Pros**:
- Off-site backup
- Collaboration with team members
- Access from multiple devices
- Free hosting (GitHub, GitLab)

**Cons**:
- Requires manual setup
- Potential security risk if repo is public
- Large repos may hit size limits
- Requires Git knowledge

**Setup**:
```bash
cd /your/content/root
git remote add origin https://github.com/you/openstrand-backup.git
git branch -M main
git push -u origin main
```

**Automated push** (cron job):
```bash
# Add to crontab (every hour)
0 * * * * cd /your/content/root && git push origin main
```

## Best Practices

1. **Choose a stable content root**: Avoid cloud-synced folders (Dropbox, iCloud) to prevent conflicts
2. **Enable Git**: Version control is invaluable for recovery and history
3. **Regular backups**: Even with mirroring, maintain separate backups (Time Machine, Backblaze)
4. **Monitor storage**: Check disk space regularly, especially with large media libraries
5. **Test recovery**: Periodically verify you can restore from your backups
6. **Document your setup**: Keep notes on your storage configuration for future reference
7. **Use deduplication**: Saves significant space for media-heavy vaults
8. **Review prune behavior**: Choose the right balance between safety and convenience

## API Reference

See [Storage API Documentation](./API_STORAGE.md) for complete endpoint reference.

## See Also

- [Obsidian Vault Integration](./OBSIDIAN_VAULT.md)
- [Export & Import Guide](./EXPORTS_AND_IMPORTS.md)
- [GitHub Integration](./INTEGRATIONS_GITHUB.md)
- [Pricing Matrix](./PRICING_MATRIX.md)

