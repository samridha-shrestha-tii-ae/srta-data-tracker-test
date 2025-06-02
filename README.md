# srta-data-tracker-test

Data Tracking test with `DVC + Git + Azure`.

This repository demonstrates how to version control datasets using [DVC](), manage experiments with **Git branches**, and push/pull data to/from **Azure Blob Storage**.

- [srta-data-tracker-test](#srta-data-tracker-test)
  - [Getting Started](#getting-started)
  - [First time setup instructions for completely new repo](#first-time-setup-instructions-for-completely-new-repo)
    - [1. Git \& DVC Initialization](#1-git--dvc-initialization)
    - [2. Configure Azure Remote](#2-configure-azure-remote)
      - [Using SAS Token (Recommended)](#using-sas-token-recommended)
  - [Track and Push Data](#track-and-push-data)
  - [On Data Update](#on-data-update)
    - [Step 1: Re-track the file](#step-1-re-track-the-file)
    - [Step 2: Commit the change](#step-2-commit-the-change)
    - [Step 3: Push updated data](#step-3-push-updated-data)
  - [Branching for Experiments](#branching-for-experiments)
  - [Version Rollback](#version-rollback)
  - [Notes](#notes)

---

## Getting Started

After cloning:

```bash
# git clone repo...
pip install dvc[azure] # inside a virtual env
dvc remote modify --local azureremote sas_token "<your-sas-token>"
# if blob url is https://srtastorageaccount.blob.core.windows.net/datasets/subdata-folder?
# the storage account name is "srtastorageaccount"
dvc remote modify --local azureremote account_name "<your-storage-account-name>"
dvc pull
```

## First time setup instructions for completely new repo

### 1. Git & DVC Initialization

```bash
git init
pip install dvc[azure] # inside a virtual env
dvc init
git add .dvc .gitignore
git commit -m "Initialize DVC"
```

### 2. Configure Azure Remote

Note: any name can bechosen instead of `azureremote`. Pick one method below:

#### Using SAS Token (Recommended)

```bash
# if blob url is https://srtastorageaccount.blob.core.windows.net/datasets/subdata-folder?
# Container name and path with format is "azure://datasets/data-folder"
# And the storage account name is "srtastorageaccount"
dvc remote add -d azureremote azure://<container-name>/<path>
dvc remote modify --local azureremote sas_token "<your-sas-token>"
dvc remote modify --local azureremote account_name "<your-storage-account-name>"
```

The other options: `Using Connection String` or `Using Storage Account + Key` are provided in the [Official documentation](https://dvc.org/doc/user-guide/data-management/remote-storage/azure-blob-storage).

## Track and Push Data

```bash
dvc add data/my_dataset.csv
git add data/my_dataset.csv.dvc data/.gitignore
git commit -m "Add new dataset"
git push
dvc push
```

## On Data Update

If you edit data/my_dataset.csv after it’s been added to DVC:

### Step 1: Re-track the file

```bash
dvc add data/my_dataset.csv
# This updates the DVC file with the new version's hash.
```

### Step 2: Commit the change

```bash
git add data/my_dataset.csv.dvc
git commit -m "Update dataset after modification"
```

### Step 3: Push updated data

```bash
dvc push
DVC uploads the new version to Azure, keeping older versions safely stored.
```

## Branching for Experiments

```bash
git checkout -b data-cleaning-experiment
# Edit the dataset
dvc add data/my_dataset.csv
git commit -am "Cleaned dataset"
dvc push
```

To switch back to another version:

```bash
git checkout main
dvc checkout
```

## Version Rollback

```bash
git checkout <old-commit-hash>
dvc checkout
```
Or reset on the same branch:

```bash
git reset --hard <commit-hash>
dvc checkout
```
🧹 Clean Up Unused Cache

```bash
dvc gc -w -v
```
This removes data no longer referenced by any current version or branch.

## Notes

-   Use `.dvc/config.local` for secrets and ignore it in `.gitignore`
-   Use `dvc status` to detect data drift
-   Run `dvc pull` after cloning the repo
-   Automate `dvc push` in CI/CD for production pipelines
-   Use `dvc config core.analytics false --global` to disable analytics (Recommended)
