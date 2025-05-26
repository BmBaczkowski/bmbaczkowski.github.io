---
title: "Managing Research Data with Git Annex: Exporting to Public Repositories" 
date: 2025-06-01
lastmod: 2025-06-01
tags: ["Reproducibility", "Version Control", "Git", "Git Annex"]
author: ["Blazej M. Baczkowski"]
description: ""
summary: "How to export data using Git Annex to external data repository (e.g., OSF)"
editPost:
    URL: ""
    Text: ""
showToc: true
disableAnchoredHeadings: false

---

## Managing Large Research Datasets with Git Annex

Managing research data is one of the most demanding aspects of scientific work. Datasets and analysis outputs can be massive, frequently updated, and complex—making them difficult to version, share, or archive effectively. That’s where tools like [Git Annex](https://git-annex.branchable.com/) come in.

---

## What Is Git Annex?

[Git Annex](https://git-annex.branchable.com/) extends [Git](https://git-scm.com/) to support large files. Instead of storing these files directly in the Git repository, Git Annex manages their metadata and location, while the actual files are kept elsewhere—on external drives, cloud storage, or remote servers.

This decouples version control from data storage, allowing teams to:

- Keep repositories lightweight and fast  
- Share and collaborate without moving large datasets around  
- Maintain reproducibility through precise metadata tracking

---

## How Git Annex Works (At a Glance)

Git Annex replaces large files in your repository with symlinks. These symlinks point to actual content stored in designated remotes. The tracking information is managed in a special `git-annex` branch.

Supported remotes include:

- External hard drives  
- SSH-accessible servers  
- S3 buckets  
- And more...

This setup lets you:

- Keep your Git repo portable  
- Share metadata separately from data (e.g., for compliance or privacy)  
- Retrieve data on-demand only when necessary

---

### Example: Backing Up to an External Drive and Sharing Meta-Data via GitLab

#### Step 1: Initialize a backup remote on an external drive
```bash
git annex initremote backupdrive type=directory \
  directory=/media/drive/project \
  encryption=none
```

#### Step 2: Transfer data to the remote
```bash
git annex copy <data> --to=backupdrive
```

#### Step 3: Push metadata to GitLab
```bash
git remote add gitlab https://gitlab.com/your-username/your-repo.git
git push gitlab main
```

Don't forget to push the `git-annex` branch as well:
```bash
git push gitlab git-annex
```

🧠 **What You'll See in GitLab**:
- File symlinks instead of actual content  
- A lightweight repository  
- Reproducible metadata, enabling collaborators to fetch real data from remotes

---

## Publishing Your Data: Git Annex + OSF

When you're ready to publish your research, it's important to archive your data in a public, citable repository. One popular platform is the [Open Science Framework (OSF)](https://osf.io/).

Currently, OSF cannot serve as a Git Annex remote *directly*. However, there are workarounds using tools like:

- [`osfclient`](https://osfclient.readthedocs.io/)  
- [DataLad OSF extension](https://docs.datalad.org/projects/osf/en/latest/)

---

## 📦 Git Annex Snapshot Export

A standout feature of Git Annex is `git annex export`, which lets you export a **snapshot**—a clean, static version of your dataset—suitable for archiving and sharing.

### Snapshot Benefits

- ✅ **Clean**: no Git metadata or symlinks  
- 🔒 **Read-only**: ideal for archival  
- 📁 **Human-readable**: exports as a normal directory tree  
- 🧬 **Immutable**: tied to a specific Git commit or tag

---

## 🛠️ Export Workflow

1. **Tag the desired commit**  
2. **Initialize an export-capable remote**  
3. **Export the snapshot**

### Example

```bash
# Tag the commit you want to export
git tag -a osf-v1.0 -m "Snapshot exported to OSF (v1.0)"
git push gitlab osf-v1.0

# Initialize a remote with export capabilities
git annex initremote exportdir type=directory exporttree=yes \
  directory=~/exports/osf

# Export snapshot to the remote
git annex export osf-v1.0 --to exportdir
```

This command exports a clean, static directory tree—ideal for manual uploads or long-term storage.

---

## Exporting to OSF Manually with `osfclient`

Because Git Annex can’t push directly to OSF, use `osfclient` to upload exported files.

### Step 1: Install and authenticate

```bash
pip install osfclient
osf init
```

Configure OSF credentials in `.osfcli.config`:

```ini
[osf]
username = yourOSFaccount@example.com
project = abcDe
```

(Optional) Save your password as an environment variable:
```bash
export OSF_PASSWORD=<your_password>
```

---

### Step 2: Upload Files

`osfclient` does not support full directory uploads, so you'll need to upload recursively:

```bash
osf upload -r <local_dir> <remote_dir>
```

Once complete, deactivate the temporary remote to avoid confusion:

```bash
git annex dead <remote-name>
```

---

## 📌 Bonus Tip: Automate with DataLad

For advanced workflows, consider using [DataLad](https://www.datalad.org/) with its [OSF extension](https://docs.datalad.org/projects/osf/en/latest/). This setup integrates OSF as a remote backend, enabling push/pull operations directly—saving you from manual uploads and streamlining reproducibility across teams.

---

## Final Thoughts

Git Annex offers a powerful way to manage large datasets with reproducibility, collaboration, and efficiency. Paired with archiving platforms like OSF, it provides a robust end-to-end workflow for open, sustainable research. Snapshot exports simplify the transition from development to publication, ensuring clean and trustworthy data sharing. Higher-level tools like DataLad, built on Git Annex, further streamline versioned and reproducible data management at scale.

---
