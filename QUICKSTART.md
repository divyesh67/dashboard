# 🚀 Quick Start - Deploy Your Dashboard in 5 Minutes

## What's Ready

Your dashboard deployment package is complete:

```
dashboard/
├── index.html              ✅ Smart HTML (works locally + remotely)
├── vercel.json             ✅ Vercel configuration
├── data/                   ✅ YAML files (synced from docs/)
│   ├── bmm-workflow-status.yaml
│   └── sprint-status.yaml
├── README.md               ✅ Full documentation
└── QUICKSTART.md           ✅ This file

scripts/
└── sync-dashboard.js       ✅ Auto-sync script

package.json                ✅ Updated with sync commands
```

## 🎯 Deploy Now (3 Steps)

### Step 1: Install Dependencies

```bash
pnpm install
```

This installs `chokidar` (file watcher) needed for auto-sync.

### Step 2: Deploy to Vercel

```bash
# Install Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy (from dashboard folder)
cd dashboard
vercel --prod
```

**Follow the prompts:**
- Project name: `bmm-status-dashboard` (or your choice)
- Directory: `./` (current)

You'll get a URL like: `https://bmm-status-dashboard.vercel.app` 🎉

### Step 3: Start Auto-Sync

```bash
# From project root (not dashboard folder)
cd ..
npm run sync:dashboard
```

**You should see:**
```
╔═══════════════════════════════════════════════════════════╗
║         BMM Dashboard Auto-Sync Script                    ║
╚═══════════════════════════════════════════════════════════╝

[13:45:12] ✅ Synced 2 file(s)
[13:45:12] ✅ File watcher started
[13:45:12] ℹ️  Press Ctrl+C to stop
```

## ✅ Done!

Your dashboard is now:
- ✅ **Live** at your Vercel URL
- ✅ **Auto-updating** when you edit YAML files
- ✅ **Accessible** to your team

## 🧪 Test It

1. **Open dashboard** in browser (use Vercel URL)
2. **Edit a YAML file**:
   ```bash
   vim docs/bmm-workflow-status.yaml
   # Make a small change
   ```
3. **Watch sync script**:
   ```
   [13:46:23] ℹ️  File changed: docs/bmm-workflow-status.yaml
   [13:46:24] ✅ Synced: bmm-workflow-status.yaml
   [13:46:25] ✅ Pushed to remote - Vercel will auto-deploy
   ```
4. **Refresh dashboard** in ~30 seconds - changes appear!

## 📱 Share with Team

Just send them the URL:
```
🌐 https://your-dashboard.vercel.app
```

## 🎨 Optional: Add Custom Domain

```bash
# In Vercel dashboard:
# 1. Go to Settings → Domains
# 2. Add domain (e.g., status.yourcompany.com)
# 3. Update DNS with CNAME record
```

## 📚 Next Steps

- **Full docs**: See `DASHBOARD_DEPLOYMENT.md` for advanced setup
- **Background service**: Set up sync script to run 24/7
- **GitHub Actions**: Automate without running local script
- **Analytics**: Enable Vercel analytics

## 🆘 Troubleshooting

### Dashboard shows "Error loading data"
```bash
npm run sync:dashboard:once
git add dashboard/data/ && git commit -m "chore: sync data" && git push
```

### Sync script won't start
```bash
pnpm install  # Ensures chokidar is installed
```

### Need help?
Check `README.md` or `DASHBOARD_DEPLOYMENT.md` for detailed guides.

---

**Questions?** Everything is documented in the dashboard folder!
