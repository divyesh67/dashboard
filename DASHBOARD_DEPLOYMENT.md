# 🚀 Dashboard Deployment Guide

Complete guide to deploying your BMM Status Dashboard on Vercel.

## 📋 Table of Contents

1. [Quick Start](#-quick-start)
2. [Detailed Setup](#-detailed-setup)
3. [Automated Workflow](#-automated-workflow)
4. [Custom Domain](#-custom-domain)
5. [Troubleshooting](#-troubleshooting)
6. [Team Access](#-team-access)

---

## ⚡ Quick Start

**Get your dashboard live in 5 minutes:**

```bash
# 1. Install dependencies
pnpm install

# 2. Install Vercel CLI
npm install -g vercel

# 3. Login to Vercel
vercel login

# 4. Deploy dashboard
cd dashboard
vercel --prod

# 5. Start auto-sync (in new terminal, from project root)
npm run sync:dashboard
```

**Done!** 🎉 Your dashboard is live and will auto-update when YAML files change.

---

## 🔧 Detailed Setup

### Step 1: Install Dependencies

The sync script requires `chokidar` for file watching:

```bash
# Install all dependencies (including chokidar)
pnpm install
```

### Step 2: Prepare Dashboard Data

Copy your current YAML files to the dashboard (already done if you followed quick start):

```bash
# From project root
cp docs/bmm-workflow-status.yaml dashboard/data/
cp docs/sprint-status.yaml dashboard/data/
```

### Step 3: Deploy to Vercel

#### Option A: Vercel CLI (Fastest)

```bash
# Install Vercel CLI globally
npm install -g vercel

# Login to Vercel
vercel login
# Follow prompts in browser

# Deploy dashboard
cd dashboard
vercel

# Answer the prompts:
# ? Set up and deploy "~/dashboard"? [Y/n] y
# ? Which scope? [Your username/org]
# ? Link to existing project? [y/N] n
# ? What's your project's name? bmm-status-dashboard
# ? In which directory is your code located? ./
# ? Want to override the settings? [y/N] n

# Deploy to production
vercel --prod
```

#### Option B: GitHub + Vercel Integration

```bash
# 1. Commit dashboard to your repo
git add dashboard/
git commit -m "feat: add BMM status dashboard for Vercel deployment"
git push

# 2. Go to https://vercel.com/new
# 3. Import your GitHub repository
# 4. Configure:
#    - Framework Preset: Other
#    - Root Directory: dashboard
#    - Build Command: (leave empty)
#    - Output Directory: (leave empty)
# 5. Click "Deploy"
```

### Step 4: Set Up Auto-Sync

The auto-sync script watches your YAML files and automatically updates the dashboard:

```bash
# From project root
npm run sync:dashboard

# You should see:
# ╔═══════════════════════════════════════════════════════════╗
# ║         BMM Dashboard Auto-Sync Script                    ║
# ╚═══════════════════════════════════════════════════════════╝
# [timestamp] ℹ️  Mode: WATCH
# [timestamp] ℹ️  Auto-commit: ENABLED
# [timestamp] ✅ Synced 2 file(s)
# [timestamp] ✅ File watcher started
# [timestamp] ℹ️  Press Ctrl+C to stop
```

**What it does:**
1. ✅ Watches `docs/bmm-workflow-status.yaml` and `docs/sprint-status.yaml`
2. ✅ Copies them to `dashboard/data/` when changed
3. ✅ Auto-commits changes to git
4. ✅ Auto-pushes to GitHub
5. ✅ Vercel auto-deploys (if GitHub integration is set up)

---

## 🤖 Automated Workflow

### Daily Usage

Once set up, your workflow is simple:

1. **Edit your YAML files** as usual:
   ```bash
   # Edit workflow status
   vim docs/bmm-workflow-status.yaml

   # Edit sprint status
   vim docs/sprint-status.yaml
   ```

2. **Sync script detects changes** (if running):
   ```
   [14:23:45] ℹ️  File changed: docs/bmm-workflow-status.yaml
   [14:23:46] ✅ Synced: bmm-workflow-status.yaml
   [14:23:46] ℹ️  Committing changes...
   [14:23:47] ✅ Changes committed
   [14:23:47] ℹ️  Pushing to remote...
   [14:23:49] ✅ Pushed to remote - Vercel will auto-deploy
   ```

3. **Dashboard updates automatically** (~30-60 seconds later)

### Running Sync Script as Background Service

#### macOS/Linux (using `nohup`):
```bash
# Start in background
nohup npm run sync:dashboard > sync-dashboard.log 2>&1 &

# Check if running
ps aux | grep sync-dashboard

# View logs
tail -f sync-dashboard.log

# Stop
pkill -f sync-dashboard
```

#### macOS (using `launchd`):
```bash
# Create launch agent
cat > ~/Library/LaunchAgents/com.yourcompany.dashboard-sync.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.yourcompany.dashboard-sync</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/node</string>
        <string>/Users/YOURNAME/path/to/NDT-Inspection-Toolkit-1/scripts/sync-dashboard.js</string>
        <string>--watch</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/Users/YOURNAME/path/to/NDT-Inspection-Toolkit-1</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/dashboard-sync.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/dashboard-sync-error.log</string>
</dict>
</plist>
EOF

# Update paths in the file above, then:
launchctl load ~/Library/LaunchAgents/com.yourcompany.dashboard-sync.plist
```

#### Windows (using `pm2`):
```bash
# Install PM2 globally
npm install -g pm2

# Start sync script
pm2 start scripts/sync-dashboard.js --name dashboard-sync -- --watch

# Configure to start on boot
pm2 startup
pm2 save

# Monitor
pm2 logs dashboard-sync

# Stop
pm2 stop dashboard-sync
```

### GitHub Actions (Zero-Maintenance)

For fully automated syncing without running a local script:

Create `.github/workflows/sync-dashboard.yml`:

```yaml
name: Sync Dashboard Data

on:
  push:
    paths:
      - 'docs/bmm-workflow-status.yaml'
      - 'docs/sprint-status.yaml'
    branches:
      - main

jobs:
  sync:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Copy YAML files
        run: |
          mkdir -p dashboard/data
          cp docs/bmm-workflow-status.yaml dashboard/data/
          cp docs/sprint-status.yaml dashboard/data/

      - name: Commit and push changes
        run: |
          git config user.name "GitHub Actions Bot"
          git config user.email "actions@github.com"
          git add dashboard/data/

          if git diff --staged --quiet; then
            echo "No changes to commit"
          else
            git commit -m "chore: sync dashboard data [skip ci]"
            git push
          fi
```

**Benefits:**
- ✅ No local script needed
- ✅ Works from any machine
- ✅ Automatic on every push
- ✅ Zero maintenance

---

## 🌐 Custom Domain

### Add Your Domain

1. **In Vercel Dashboard:**
   - Go to your project → Settings → Domains
   - Click "Add Domain"
   - Enter your domain (e.g., `status.yourcompany.com`)

2. **Update DNS:**

   **Option A: CNAME (subdomain)**
   ```
   Type: CNAME
   Name: status
   Value: cname.vercel-dns.com
   ```

   **Option B: A Record (root domain)**
   ```
   Type: A
   Name: @
   Value: 76.76.21.21
   ```

3. **SSL Certificate:**
   - Automatically provisioned by Vercel
   - Usually takes 1-2 minutes

### Multiple Domains

You can add multiple domains to the same dashboard:

```
- status.yourcompany.com (primary)
- dashboard.yourcompany.com
- bmm.yourcompany.com
```

All will point to the same deployment.

---

## 🛠️ Troubleshooting

### Dashboard Shows "Error loading data"

**Problem:** YAML files not found in `/data/` directory

**Solution:**
```bash
# Sync files manually
npm run sync:dashboard:once

# Verify files exist
ls -la dashboard/data/

# Commit and push
git add dashboard/data/
git commit -m "chore: add dashboard data files"
git push
```

### Sync Script Won't Start

**Problem:** `chokidar` not installed

**Solution:**
```bash
# Install dependencies
pnpm install

# Or install chokidar specifically
pnpm add -D chokidar
```

### Changes Not Appearing on Dashboard

**Possible causes:**

1. **Browser cache:**
   - Hard refresh: `Cmd+Shift+R` (Mac) or `Ctrl+Shift+R` (Windows)
   - Or open in incognito mode

2. **Sync script not running:**
   ```bash
   # Check if sync script is running
   ps aux | grep sync-dashboard

   # Restart if needed
   npm run sync:dashboard
   ```

3. **Git push failed:**
   ```bash
   # Check git status
   cd dashboard
   git status

   # Push manually
   git push
   ```

4. **Vercel deployment failed:**
   - Check deployment logs in Vercel dashboard
   - Look for build errors

### Vercel Deployment Fails

**Problem:** Wrong root directory configured

**Solution:**
1. Go to Vercel → Project Settings → General
2. Set **Root Directory**: `dashboard`
3. Set **Build Command**: (leave empty)
4. Set **Output Directory**: (leave empty)
5. Click "Save"
6. Trigger new deployment

### Permission Denied on Sync Script

**Problem:** Script not executable

**Solution:**
```bash
chmod +x scripts/sync-dashboard.js
```

---

## 👥 Team Access

### Share Dashboard with Team

Once deployed, just share the URL:

```
🌐 Dashboard URL: https://bmm-status-dashboard.vercel.app
```

Or with custom domain:

```
🌐 Dashboard URL: https://status.yourcompany.com
```

### Password Protection (Optional)

**Option 1: Vercel Password Protection**

1. Go to Project Settings → Deployment Protection
2. Enable "Password Protection"
3. Set password
4. Share password with team

**Option 2: Environment-Based Auth**

Add to `vercel.json`:

```json
{
  "env": {
    "PROTECTION_BYPASS": "secret-password-here"
  }
}
```

Then add middleware in dashboard for auth check.

### Read-Only Access

The dashboard is inherently read-only:
- ✅ No write capabilities
- ✅ No API endpoints
- ✅ Pure static HTML/CSS/JS
- ✅ Safe to share widely

---

## 📊 Monitoring & Analytics

### View Deployment Logs

```bash
# Using Vercel CLI
vercel logs [deployment-url]

# Or in Vercel dashboard
# Project → Deployments → [Select deployment] → Logs
```

### Add Analytics

Vercel provides built-in analytics:

1. Go to Project Settings → Analytics
2. Enable Web Analytics
3. Get insights on:
   - Page views
   - Unique visitors
   - Referrers
   - Device types

### Custom Analytics

Add to `dashboard/index.html` before `</body>`:

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

---

## 🔄 Updates & Maintenance

### Update Dashboard HTML

```bash
# Edit dashboard/index.html
vim dashboard/index.html

# Commit and push
git add dashboard/index.html
git commit -m "feat: update dashboard styling"
git push

# Vercel auto-deploys
```

### Update Sync Script

```bash
# Edit sync script
vim scripts/sync-dashboard.js

# Restart sync script
npm run sync:dashboard
```

### Roll Back Deployment

```bash
# Using Vercel CLI
vercel rollback [deployment-url]

# Or in Vercel dashboard
# Deployments → [Previous deployment] → Promote to Production
```

---

## 📚 Additional Resources

- [Dashboard README](README.md) - Full dashboard documentation
- [Sync Script](../scripts/sync-dashboard.js) - Auto-sync implementation
- [Vercel Documentation](https://vercel.com/docs)
- [Vercel CLI Reference](https://vercel.com/docs/cli)

---

## 🎉 Success Checklist

Before considering setup complete:

- [ ] Dashboard deployed to Vercel
- [ ] Custom domain configured (optional)
- [ ] YAML files synced to `dashboard/data/`
- [ ] Sync script running in background
- [ ] Dashboard accessible from web browser
- [ ] Team members can access dashboard
- [ ] Auto-update working (test by editing YAML file)
- [ ] SSL certificate active (https://)

**All checked?** Congratulations! 🎊 Your dashboard is production-ready.

---

## 💡 Pro Tips

1. **Bookmark the dashboard** for quick access
2. **Add to Slack** - Share the URL in your team channel
3. **Mobile-friendly** - Access from phone/tablet
4. **Auto-refresh** - Dashboard refreshes every 10 minutes automatically
5. **Real-time updates** - Enable browser notifications for changes
6. **Monitor uptime** - Use UptimeRobot or similar to monitor dashboard availability

---

**Need help?** Check the [Troubleshooting](#-troubleshooting) section or create an issue.
