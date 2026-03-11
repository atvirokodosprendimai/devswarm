# Self-Distributing Swarm: Cross-Platform Implementation

**Write once, swarm anywhere.**

A Git repository that distributes and replicates itself across any Git forge using only standard Git features and CI/CD.

Read the story continuation: [STORY.md](STORY.md)

## Quick Start

### Join an Existing Swarm (Recommended)

1. **Fork** this repository (click the Fork button)
2. **Enable CI/CD** (GitHub: Actions tab → "I understand my workflows, go ahead and enable them")
3. **Done** - you're now part of the swarm

That's it. The repo already contains everything needed. No install script required.

### Try Pair Programming (New!)

1. **Open** `pair-editor.html` in your browser
2. **Connect** to `wss://demos.yjs.dev` with room `devswarm-test-123`
3. **Share** the room name with others
4. **Code together** in real-time

See [PAIR_PROGRAMMING.md](PAIR_PROGRAMMING.md) for full guide.

### Add Swarm to Your Own Repo

```bash
# Clone any fork and run the installer
git clone https://github.com/HackrsValv/devswarm-cross-platform
./devswarm-cross-platform/install-swarm.sh /path/to/your-repo
```

This copies the coordinator and CI configs into your repository.

## How It Works

### The Breakthrough

**The repository IS the distributable unit.**

```
Traditional:            Self-Distributing Swarm:
├── Code                ├── Code
├── Docker              ├── Coordinator script
└── Deploy script       ├── CI/CD config
                        └── .swarm/
                              └── manifest.json
                        
Deploy = build+push     Deploy = git push
Scale = more servers    Scale = more forks
Discovery = DNS         Discovery = GitHub API
```

### Three Primitives

1. **Fork = Replication**
   - Click "Fork" button
   - Get complete copy instantly
   - No build, no deploy, no config

2. **CI/CD = Coordination**
   - Runs automatically every 6 hours
   - Syncs with upstream
   - Updates swarm topology
   - Self-healing

3. **Git = Consensus**
   - Merge conflicts = consensus mechanism
   - Git history = audit trail
   - Branches = experimentation
   - Forks = true copies

## Platform Support

| Platform | Fork API | CI/CD | Status |
|----------|----------|-------|--------|
| GitHub | ✅ REST | Actions | ✅ Full |
| GitLab | ✅ REST | GitLab CI | ✅ Full |
| Gitea | ✅ REST | Actions | ✅ Full |
| Forgejo | ✅ REST | Actions | ✅ Full |
| Codeberg | ✅ (Forgejo) | Actions | ✅ Full |
| Woodpecker | ⚠️ Manual | Woodpecker | ✅ Full |
| Sourcehut | ⚠️ Manual | builds.sr.ht | 🚧 WIP |
| Generic Git | ⚠️ Branches | Manual | ✅ Basic |

### Works Everywhere

The same coordinator script runs on:
- GitHub Actions
- GitLab CI
- Gitea Actions  
- Forgejo Actions
- Woodpecker CI
- Any CI with bash+git+curl+jq

## Architecture

### Components

```
swarm-coordinator.sh          Universal coordinator (platform-agnostic)
├── Platform detection        (GitHub? GitLab? Gitea? Generic?)
├── Fork discovery            (API or git branches)
├── Health calculation        (degraded/vulnerable/stable/healthy)
├── Manifest generation       (.swarm/manifest.json)
└── Upstream sync             (merge latest changes)

.github/workflows/swarm.yml   GitHub Actions config
.gitlab-ci.yml                GitLab CI config
.gitea/workflows/swarm.yml    Gitea/Forgejo config
.woodpecker.yml               Woodpecker CI config
```

### Swarm Manifest

```json
{
  "version": "1.0.0",
  "platform": "github",
  "repository": "alice/project",
  "swarm_topology": {
    "fork_count": 12,
    "health": "healthy",
    "forks": [
      "bob/project",
      "charlie/project",
      "diana/project"
    ]
  },
  "distribution_mechanics": {
    "method": "fork-native",
    "replication": "automatic via git clone",
    "discovery": "platform API + git branches",
    "healing": "via CI/CD sync",
    "consensus": "git merge"
  }
}
```

### Health Levels

- **Healthy** (10+ forks): High redundancy, excellent availability
- **Stable** (6-9 forks): Good redundancy, reliable
- **Vulnerable** (3-5 forks): Minimal redundancy, at risk
- **Degraded** (<3 forks): Critical, needs more forks

## Installation

### Option 1: Fork (Recommended)

Just fork this repo. The CI/CD configs are already included.

| Platform | After Forking |
|----------|---------------|
| GitHub | Go to Actions tab → Enable workflows |
| GitLab | CI runs automatically |
| Gitea/Forgejo | Enable Actions if required by instance |
| No CI | Run `./swarm-coordinator.sh` manually or via cron |

### Option 2: Add Swarm to Existing Repo

```bash
# Clone from any fork in the swarm
git clone https://github.com/HackrsValv/devswarm-cross-platform

# Run installer pointing to your repo
./devswarm-cross-platform/install-swarm.sh /path/to/your-repo

# Commit and push
cd /path/to/your-repo
git add -A
git commit -m "🐝 Initialize swarm"
git push
```

### Option 3: Manual Setup

```bash
# Copy coordinator script
cp swarm-coordinator.sh /path/to/your-repo/
chmod +x /path/to/your-repo/swarm-coordinator.sh

# Copy CI config for your platform
# GitHub:
cp templates/github-actions.yml /path/to/your-repo/.github/workflows/swarm.yml

# GitLab:
cp templates/gitlab-ci.yml /path/to/your-repo/.gitlab-ci.yml

# Gitea/Forgejo:
cp templates/gitea-actions.yml /path/to/your-repo/.gitea/workflows/swarm.yml
```

> **No central server**: Bootstrap from any fork. The swarm IS the distribution network.

## Usage

### Join a Swarm

```bash
# Just fork the repo (or click Fork button)
# CI/CD runs automatically
# You're now part of the swarm
```

### Check Swarm Health

```bash
cat .swarm/manifest.json | jq .

# Or via GitHub API
curl https://api.github.com/repos/alice/project/forks | jq 'length'
```

### Trigger Manual Sync

```bash
# GitHub: Actions tab → Run workflow
# GitLab: CI/CD → Run pipeline  
# Gitea: Actions tab → Run workflow
```

### Migrate Platforms

```bash
# Move from GitHub to GitLab
git remote add gitlab https://gitlab.com/alice/project.git
git push gitlab main

# Update CI config
rm -rf .github/
cp .gitlab-ci.yml.template .gitlab-ci.yml
git add -A
git commit -m "Migrate to GitLab"
git push gitlab main

# Swarm reconstitutes on GitLab
# Old forks still work via git protocol
```

## Why This Works

### Zero Infrastructure

- **Storage**: Git forge pays
- **Compute**: Free CI/CD minutes
- **Network**: Git forge's CDN  
- **Discovery**: Platform API (free)

### Censorship Resistance

To kill the swarm, you must:
1. Delete original repo
2. Find ALL forks (could be thousands)
3. Delete each fork (requires access)
4. Hope no one made offline clones
5. Prevent anyone from re-uploading

Good luck with that. 🐝

### Self-Healing

```
Original repo deleted?
├── Any fork becomes new origin
├── Others PR to new canonical
├── Fork graph updates naturally
└── Swarm continues

Fork becomes stale?
├── CI/CD detects lag
├── Fetches upstream
├── Auto-merges changes
└── Fork catches up

Network partition?
├── Each partition continues
├── Eventually reconnects
├── Git merge resolves conflicts
└── Swarm reunifies
```

## Philosophy

Extends [Unhosted](https://unhosted.org) to code:

| Unhosted Web | Self-Distributing Swarm |
|--------------|-------------------------|
| Users own data | Users own copies (via fork) |
| Apps are interfaces | Code distributes itself |
| Storage user-controlled | Hosting decentralized |
| No platform lock-in | Works on any Git forge |

### Principles

1. **Transparent**: No hidden mechanisms
2. **Consensual**: Explicit fork action
3. **Decentralized**: No single point of failure
4. **Portable**: Works on any Git host
5. **Free**: Uses existing infrastructure

## Advanced Usage

### Custom Coordination Logic

```bash
# Edit swarm-coordinator.sh
vim swarm-coordinator.sh

# Add custom logic
custom_logic() {
    # Your code here
    # Examples:
    # - Alert on low health
    # - Auto-create issues
    # - Update documentation
    # - Trigger deployments
}

# Call in main()
custom_logic
```

### Multi-Platform Swarm

```bash
# Fork spans multiple platforms
GitHub fork → GitLab fork → Gitea fork

# Each platform discovers others via git remotes
git remote add github https://github.com/alice/project
git remote add gitlab https://gitlab.com/alice/project  
git remote add gitea https://gitea.io/alice/project

# Coordinator syncs all remotes
for remote in github gitlab gitea; do
    git fetch $remote
    git merge $remote/main --no-edit
done
```

### Private Swarms

```bash
# Works with private repos too
# Fork = invite collaborator
# Still self-distributing within team
# Still self-healing
# Still zero infrastructure

# Each team member gets full copy
# No central server required
```

## Comparison

### vs Docker Swarm

| Feature | Docker Swarm | Git Swarm |
|---------|--------------|-----------|
| Distribution | Docker registry | Git fork |
| Orchestration | Swarm manager | CI/CD |
| Discovery | DNS | Platform API |
| Scaling | Add nodes | Add forks |
| Infrastructure | Required | Zero |
| Setup time | Hours | Seconds |

### vs Kubernetes

| Feature | Kubernetes | Git Swarm |
|---------|------------|-----------|
| Complexity | High | Minimal |
| Learning curve | Steep | Gentle |
| Cost | Significant | Free |
| Vendor lock-in | Risk | None |
| Portability | Limited | Universal |

### vs IPFS

| Feature | IPFS | Git Swarm |
|---------|------|-----------|
| Content addressing | ✅ | Git commits |
| DHT | ✅ | Platform API |
| Pinning required | ✅ | No (forks are pins) |
| Browser support | Limited | Full (via forge) |
| Mainstream adoption | Growing | Established |

## Technical Details

### Platform Detection

```bash
detect_platform() {
    # Check CI env vars first
    if [ -n "$GITHUB_ACTIONS" ]; then echo "github"
    elif [ -n "$GITLAB_CI" ]; then echo "gitlab"
    elif [ -n "$GITEA_ACTIONS" ]; then echo "gitea"
    # ... etc
    
    # Fall back to git remote parsing
    REMOTE=$(git remote get-url origin)
    case "$REMOTE" in
        *github.com*) echo "github" ;;
        *gitlab.com*) echo "gitlab" ;;
        # ... etc
    esac
}
```

### Fork Discovery APIs

```bash
# GitHub
curl https://api.github.com/repos/owner/repo/forks

# GitLab  
curl https://gitlab.com/api/v4/projects/owner%2Frepo/forks

# Gitea/Forgejo
curl https://gitea.io/api/v1/repos/owner/repo/forks

# Generic (via git branches)
git ls-remote origin 'refs/heads/swarm/fork-*'
```

### Upstream Sync

```bash
# Detect if this is a fork
UPSTREAM=$(curl api/repos/owner/repo | jq -r '.parent.clone_url')

if [ -n "$UPSTREAM" ]; then
    git remote add upstream "$UPSTREAM"
    git fetch upstream
    git merge upstream/main --no-edit
fi
```

## Troubleshooting

### CI/CD not running?

```bash
# GitHub: Check Actions tab
# GitLab: Check CI/CD → Pipelines
# Gitea: Check Actions tab

# Verify workflow file exists
ls .github/workflows/swarm.yml  # GitHub
ls .gitlab-ci.yml               # GitLab
ls .gitea/workflows/swarm.yml   # Gitea
```

### Forks not discovered?

```bash
# Check API rate limits
curl -i https://api.github.com/rate_limit

# Use authenticated requests
export GITHUB_TOKEN=your_token
./swarm-coordinator.sh

# Manual discovery
git ls-remote https://github.com/owner/repo
```

### Merge conflicts?

```bash
# CI/CD will skip auto-merge
# Manual resolution required
git fetch upstream
git merge upstream/main
# Resolve conflicts
git commit
git push
```

## Formal Security Specification

The `formal-spec/` directory contains a [Tamarin Prover](https://tamarin-prover.github.io/)
security protocol theory (`devswarm.spthy`) that formally models the
DevSwarm coordination protocol and proves its key security properties:

| Property | Description |
|----------|-------------|
| **Fork Ancestry** | Every swarm node traces back to the origin through a chain of legitimate forks — no phantom nodes |
| **Sync Integrity** | A fork only syncs commits from its declared upstream parent |
| **Platform API Integrity** | Fork listings from the platform reflect genuine, registered forks |
| **Manifest Authenticity** | The swarm manifest lists only platform-verified forks |
| **Swarm Membership Integrity** | Every manifest entry is a legitimately created fork node |

```bash
# Prove all security properties:
tamarin-prover formal-spec/devswarm.spthy --prove

# Interactive exploration:
tamarin-prover interactive formal-spec/devswarm.spthy
```

See [formal-spec/README.md](formal-spec/README.md) for full documentation.

## Roadmap

- [x] Cross-platform support (tool #3) ✅ DONE
- [x] Bootstrap CLI (tool #1) ✅ DONE
- [x] Visualization dashboard (tool #2) ✅ DONE
- [x] Pair programming editor ✅ DONE (2025-12-12)
- [x] Formal Tamarin specification ✅ DONE
- [ ] IPFS fragment storage ← NEXT (Week 1)
- [ ] Erasure coding (k=6, m=3) ← NEXT (Week 1)
- [ ] Age encryption for fragments ← NEXT (Week 1)
- [ ] Multi-backend storage (IPFS + GitHub + S3) (Week 2)
- [ ] CRDT agent coordination (Week 3)
- [ ] Auto-migration on platform failure (Month 2)

**Current Phase:** Pair programming + IPFS integration
**See:** [TODO.md](TODO.md) for detailed task list

## License

MIT - Fork freely 🐝

## Contributing

This repo practices what it preaches:

```bash
# Fork this repo
# Make changes
# Push to your fork
# Open PR

# Your fork is already:
# - A complete copy
# - Part of the swarm  
# - Self-distributing
# - Self-healing
```

## Philosophy

> "The best way to distribute software is to make the software distribute itself."

Traditional distribution requires infrastructure. Self-distributing swarms require only:
- Git (established 2005)
- Forks (basic Git feature)
- CI/CD (free on all platforms)

That's it. No servers, no containers, no orchestration, no cost.

**The repository IS the infrastructure.**

---

## Known Forks

Bootstrap from any of these. This list updates as the swarm syncs.

<!-- SWARM_FORKS_START -->
| Fork | Platform | Status |
|------|----------|--------|
| [HackrsValv/devswarm](https://github.com/HackrsValv/devswarm) | Github | Origin |
| [atvirokodosprendimai/devswarm](https://github.com/atvirokodosprendimai/devswarm) | Github | Fork |
| [nycterent/devswarm](https://github.com/nycterent/devswarm) | Github | Fork |
<!-- SWARM_FORKS_END -->

> **Your fork not listed?** It will appear after the next swarm sync cycle (runs every 6 hours).
> Seeing your fork here = the swarm is working.

---

🐝 **By reading this, you're already participating in the swarm.** 🐝

If this repo disappears, clone from any fork listed above. The swarm persists.
