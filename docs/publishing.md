# Publishing checklist

A short release runbook for turning the local repository into a public portfolio piece. Work from the
repository root and stop if a preflight check fails — publishing a broken README or missing `.pbix` is
harder to undo than fixing it locally.

## At a glance

| Setting | Value |
|---|---|
| Repository | `beanstream-powerbi` |
| GitHub owner | `malikb0` |
| Visibility | Public |
| Default branch | `main` |
| Licence | MIT |
| Description | Three-page Power BI KPI dashboard over a PostgreSQL star schema — Power Query, 16 DAX measures, reconciled to source KPIs. |
| Topics | `power-bi` · `dax` · `power-query` · `data-analytics` · `postgresql` · `portfolio` · `business-intelligence` |
| Social preview | `screenshots/01-executive.png` |

## 1. Preflight

- [ ] The report is saved at `powerbi/beanstream-kpi.pbix`.
- [ ] The three screenshots exist and load: `01-executive.png`, `02-trends.png`, `03-products.png`.
- [ ] The reconciliation in [Build guide §12](spec.md#12-reconcile) still passes.
- [ ] `README.md` renders, images load, and the Mermaid syntax is valid.
- [ ] `git status` shows only intentional changes.
- [ ] No credentials, `.env` files, database dumps, or raw customer data are staged.
- [ ] The docs are current: [documentation hub](README.md), [architecture](architecture.md),
      [data model](data-model.md), [measures](measures.md), and [data dictionary](../data/README.md).

> **Do not initialise the GitHub repo with a README or licence.** Both already exist here; creating
> them remotely produces a conflicting first commit.

## 2. Commit

From the repository root:

```bash
git status
git add .
git commit -m "Build BeanStream Power BI KPI dashboard"
```

If the history is already committed, skip the commit and confirm the working tree is clean.

## 3. Create the GitHub repository

With the GitHub CLI:

```bash
gh repo create malikb0/beanstream-powerbi --public --source=. --remote=origin --push
```

Or create it in the browser — public, with **no** README or licence — then connect the existing local
history:

```bash
git remote add origin https://github.com/malikb0/beanstream-powerbi.git
git push -u origin main
```

## 4. Verify the publication

Open the repository in a private window, where no GitHub login cache can hide a permissions problem.

- [ ] The README renders and the three screenshots load.
- [ ] `powerbi/beanstream-kpi.pbix` downloads and opens.
- [ ] Relative links work: docs, screenshots, licence, and notice.
- [ ] Mermaid renders on GitHub — especially the architecture diagram.
- [ ] The repository description and topics are set.
- [ ] The social preview is `screenshots/01-executive.png`.
- [ ] The default branch is `main`.

## 5. Repository settings

| Setting | Value |
|---|---|
| Description | Three-page Power BI KPI dashboard over a PostgreSQL star schema — Power Query, 16 DAX measures, reconciled to source KPIs. |
| Topics | `power-bi`, `dax`, `power-query`, `data-analytics`, `postgresql`, `portfolio`, `business-intelligence` |
| Social preview | Upload `screenshots/01-executive.png` |
| Default branch | `main` |
| Pin | Pin the repo on the profile after publishing |

Keep the synthetic-data note visible: nothing in this repository is real customer data. Do not commit
credentials; the local connection uses the platform repo's defaults.

## 6. If something fails

| Problem | Fix |
|---|---|
| `gh: command not found` | Install the GitHub CLI, or use the browser path in §3 |
| `remote origin already exists` | `git remote -v`; update it with `git remote set-url origin <url>` |
| Push rejected | `git pull --rebase origin main`, resolve, then push again |
| README images broken | Check the relative path and exact filename in `screenshots/` |
| Social preview not updating | GitHub caches previews; re-upload and allow a few minutes |
| Mermaid shows as code | Check the fence is `mermaid` and the diagram has no syntax error |
| `.pbix` too large | Remove test pages from the report; use Git LFS only if genuinely necessary |

## Definition of done

- [ ] Public repository renders cleanly for a logged-out visitor.
- [ ] Report, screenshots, and reconciliation are present.
- [ ] Documentation is linked from the README and works from a fresh clone.
- [ ] Description, topics, and social preview are set.
- [ ] Repository is pinned on the profile.
- [ ] If the repository name changes, update the case-study and README links.
- [ ] Optional: publish the report to the Power BI service and add the live link.
