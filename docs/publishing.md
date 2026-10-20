# Publishing checklist

| Setting | Value |
|---|---|
| Repo | `beanstream-powerbi` |
| Description | Three-page Power BI KPI dashboard over a PostgreSQL star schema — Power Query, 16 DAX measures, reconciled to source KPIs. |
| Topics | `power-bi` · `dax` · `power-query` · `data-analytics` · `postgresql` · `portfolio` · `business-intelligence` |
| Social preview | `screenshots/01-executive.png` |

## 1. Commit

From the repo root:

```bash
git status
git add .
git commit -m "Build BeanStream Power BI KPI dashboard"
```

## 2. Create the GitHub repo

With the GitHub CLI:

```bash
gh repo create malikb0/beanstream-powerbi --public --source=. --remote=origin --push
```

Or create it in the browser — public, and **do not** initialise it with a README or licence, because the
repo already has both — then:

```bash
git remote add origin https://github.com/malikb0/beanstream-powerbi.git
git push -u origin main
```

## 3. After publishing

- [ ] The README renders and the three screenshots load.
- [ ] `powerbi/beanstream-kpi.pbix` is present.
- [ ] Description and topics are set.
- [ ] Social preview is `screenshots/01-executive.png`.
- [ ] The repo is pinned on the profile.
- [ ] If the repo name differs, update the case-study link.
- [ ] Optional: publish the report to the Power BI service and add the link.

## 4. Repository settings

- Default branch: `main`.
- Keep the synthetic data note visible — nothing here is real customer data.
- Do not commit credentials; the local connection uses the platform repo's defaults.
