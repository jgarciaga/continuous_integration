# Automated PR Reviewer Assignment Guide

## Overview

This guide explains how to simulate a pull request (PR) that requires approval from two GitHub accounts on a single file change. In addition, it demonstrates how to automate the reviewer assignment process using GitHub Actions so that you don't have to manually assign Account A and Account B every time.

## Prerequisites

- Two GitHub accounts (e.g., Account A and Account B)
- Administrative access to a GitHub repository
- Familiarity with GitHub Actions and basic YAML configuration

## Step-by-Step Guide

### 1. Prepare Your Repository
- **Create or Choose a Repository:**  
  Use Account A to create a new repository or select an existing one.
- **Add the Second Account as a Collaborator:**  
  In your repository settings, add Account B as a collaborator to grant it review permissions.

### 2. Set Up Branch Protection Rules
- Navigate to **Settings > Branches** in your repository.
- Create a branch protection rule for your default branch (typically `main` or `master`):
  - Enable **Require pull request reviews before merging.**
  - Set the **Required approving reviews** to **2**.
  
This ensures that a PR must be approved by two separate accounts before it can be merged.

### 3. Create a Feature Branch and Make Changes
- **Clone the Repository:**  
  Clone the repository locally or use GitHub’s online editor.
- **Create a New Branch:**  
  Create a branch (e.g., `feature/update-file`) to work on your change.
- **Edit a File:**  
  Make a change to a file (e.g., update the `README.md`).
- **Commit and Push:**  
  Commit your changes and push the branch to GitHub.

### 4. Automate Reviewer Assignment Using GitHub Actions

Instead of manually assigning reviewers, you can set up a GitHub Action to automatically request reviews from both accounts.

#### Setting Up the GitHub Action

1. **Create a Workflow File:**  
   In your repository, create a file at `.github/workflows/auto_assign_reviewers.yml`.

2. **Add the Following YAML Configuration:**

   ```yaml
   name: Auto Assign Reviewers

   on:
     pull_request:
       types: [opened]

   jobs:
     assign-reviewers:
       runs-on: ubuntu-latest
       steps:
         - name: Request reviewers
           uses: actions/github-script@v6
           with:
             github-token: ${{ secrets.GITHUB_TOKEN }}
             script: |
               // Define the reviewers - replace with your actual GitHub usernames
               const reviewers = ["accountA", "accountB"];
               const prNumber = context.issue.number;
               await github.pulls.requestReviewers({
                 owner: context.repo.owner,
                 repo: context.repo.repo,
                 pull_number: prNumber,
                 reviewers: reviewers,
               });
