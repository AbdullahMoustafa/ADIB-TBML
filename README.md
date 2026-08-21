# ADIB Trade Lane Watch

Static HTML visualization that reads `ship_movements_bl.csv` from the site root.

Deployment
 - GitHub: push this repository to GitHub (already done).
 - Netlify: link this GitHub repository in the Netlify UI for automatic deploys, or use the CLI:

```bash
# install (once)
npm install -g netlify-cli

# login (opens browser)
netlify login

# deploy (production):
netlify deploy --prod --dir="c:\Users\AbdullahMous_z7z8mg7\OneDrive - Data Science Middle East FZCO\Desktop\ADIB"
```

Notes
- If you upload `ship_movements_bl.csv` to a host make sure its `Content-Type` is `text/csv` for some strict CDNs.
- Netlify's GitHub integration will automatically build and publish from the default branch.
