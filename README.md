# BMM Status Dashboard - Vercel Deployment

This directory contains the production-ready dashboard for hosting on Vercel.

## 🌐 Live Dashboard

Once deployed, your dashboard will:
- **Auto-update** when you push changes to GitHub
- **Work anywhere** - accessible from any device
- **Share with team** - just send them the URL
- **Stay fast** - powered by Vercel's CDN

## 📁 Directory Structure

```
dashboard/
├── index.html           # Main dashboard (auto-detects local vs production)
├── vercel.json          # Vercel configuration
├── data/                # YAML data files (synced from ../docs/)
│   ├── bmm-workflow-status.yaml
│   └── sprint-status.yaml
└── README.md            # This file
```

## 🚀 Quick Start

### Option 1: Git Push Auto-Deploy (Recommended)

1. **Initial Setup** (one-time)
   ```bash
   # Install Vercel CLI globally
   npm install -g vercel

   # Login to Vercel
   vercel login

   # Deploy from dashboard directory
   cd dashboard
   vercel

   # Follow prompts:
   # - Link to existing project? No
   # - Project name: bmm-status-dashboard (or your choice)
   # - Directory: ./ (current directory)

   # Connect to Git for auto-deployments
   vercel --prod
   ```

2. **Start Auto-Sync** (keeps dashboard updated)
   ```bash
   # From project root
   npm run sync:dashboard

   # This will:
   # ✅ Watch your YAML files for changes
   # ✅ Auto-copy them to dashboard/data/
   # ✅ Auto-commit and push to GitHub
   # ✅ Trigger Vercel auto-deploy
   ```

3. **That's it!**
   - Make changes to `docs/bmm-workflow-status.yaml` or `docs/sprint-status.yaml`
   - Script auto-syncs → Git auto-commits → Vercel auto-deploys
   - Team sees updates in ~30-60 seconds

### Option 2: Manual Vercel CLI Deployment

```bash
# One-time deploy
cd dashboard
vercel --prod

# Sync files manually when needed
npm run sync:dashboard:once
```

### Option 3: Connect GitHub → Vercel (Zero-maintenance)

1. **Push dashboard to GitHub**
   ```bash
   git add dashboard/
   git commit -m "feat: add dashboard deployment"
   git push
   ```

2. **Connect to Vercel**
   - Go to [vercel.com](https://vercel.com)
   - Click "Add New Project"
   - Import your GitHub repository
   - **Root Directory**: Set to `dashboard`
   - Click "Deploy"

3. **Enable Auto-Deployments**
   - Settings → Git → Enable "Production Branch" (main)
   - Now every push to `main` auto-deploys

4. **Run Sync Script**
   ```bash
   npm run sync:dashboard
   ```
   - Watches YAML files
   - Auto-commits changes
   - Vercel auto-deploys on push

## 🛠️ Configuration

### Custom Domain

1. **Add domain in Vercel**:
   - Project Settings → Domains
   - Add your domain (e.g., `status.yourcompany.com`)

2. **Update DNS**:
   - Add CNAME record: `status` → `cname.vercel-dns.com`
   - Or use Vercel nameservers

3. **SSL**: Automatic via Vercel

### Environment-Specific Behavior

The dashboard automatically detects its environment:

- **Local Development** (`localhost`):
  - Reads from `../docs/*.yaml`
  - Hot reload on file changes
  - Debug logging enabled

- **Production** (your domain):
  - Reads from `/data/*.yaml`
  - Caches disabled for real-time updates
  - Optimized performance

### Sync Script Options

```bash
# Watch mode (default) - runs continuously
npm run sync:dashboard
node scripts/sync-dashboard.js --watch

# One-time sync
npm run sync:dashboard:once
node scripts/sync-dashboard.js --once

# Sync + Deploy immediately
node scripts/sync-dashboard.js --deploy
```

### Advanced: GitHub Actions Auto-Sync

For zero-maintenance syncing, add this workflow:

`.github/workflows/sync-dashboard.yml`:
```yaml
name: Sync Dashboard

on:
  push:
    paths:
      - 'docs/bmm-workflow-status.yaml'
      - 'docs/sprint-status.yaml'

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Copy YAML files
        run: |
          mkdir -p dashboard/data
          cp docs/bmm-workflow-status.yaml dashboard/data/
          cp docs/sprint-status.yaml dashboard/data/
      - name: Commit changes
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add dashboard/data/
          git commit -m "chore: sync dashboard data" || exit 0
          git push
```

## 🔍 Troubleshooting

### Dashboard shows "Error loading data"

**Cause**: YAML files missing in `/data/` folder

**Fix**:
```bash
npm run sync:dashboard:once
git add dashboard/data/
git commit -m "chore: add initial dashboard data"
git push
```

### Sync script can't find files

**Cause**: Running from wrong directory

**Fix**: Always run from project root:
```bash
cd /path/to/NDT-Inspection-Toolkit-1
npm run sync:dashboard
```

### Vercel deployment fails

**Cause**: Wrong root directory

**Fix**: In Vercel settings:
- Root Directory: `dashboard`
- Build Command: (leave empty)
- Output Directory: (leave empty)

### Changes not appearing

**Cause**: Browser cache

**Fix**: Hard refresh (Cmd+Shift+R or Ctrl+Shift+R)

The dashboard has `Cache-Control: no-cache` headers, but browsers can be stubborn.

## 📊 Features

- ✅ **Real-time Status**: Auto-refreshes every 10 minutes
- ✅ **Offline Support**: Works without your dev machine running
- ✅ **Team Collaboration**: Share URL with anyone
- ✅ **Mobile Friendly**: Responsive design
- ✅ **Zero Maintenance**: Auto-deploys on git push
- ✅ **Fast Loading**: Optimized for performance
- ✅ **Secure**: Read-only dashboard (no write access)

## 🎨 Customization

### Update Project Name

Edit `dashboard/index.html` line 854:
```html
<span class="subtitle">Project: YOUR_PROJECT_NAME</span>
```

### Change Refresh Interval

Edit `dashboard/index.html` line 1696:
```javascript
setInterval(loadData, 600000); // 600000ms = 10 minutes
```

### Add Password Protection

Use Vercel's password protection:
```json
// vercel.json
{
  "functions": {
    "api/**/*.js": {
      "headers": {
        "WWW-Authenticate": "Basic"
      }
    }
  }
}
```

Or use edge middleware for custom auth.

## 📝 Notes

- **No Backend Required**: Pure static HTML/CSS/JS
- **No Build Step**: Deploys instantly
- **Zero Cost**: Free on Vercel's hobby plan
- **YAML Parsing**: Uses `js-yaml` from CDN (no npm install)
- **Git-Powered**: Your GitHub repo is the source of truth

## 🆘 Support

If you encounter issues:

1. Check Vercel deployment logs
2. Verify YAML files are in `dashboard/data/`
3. Test locally: `open dashboard/index.html`
4. Check browser console for errors

## 🔗 Links

- [Vercel Documentation](https://vercel.com/docs)
- [Dashboard Source](./index.html)
- [Sync Script](../scripts/sync-dashboard.js)
