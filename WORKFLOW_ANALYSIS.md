# Workflow Analysis: deploy.yml

## What triggers this workflow to run?
The workflow is triggered by two events on the `on:` section:
- A `push` to the `main` branch
- A `pull_request` targeting the `main` branch

## What are the four main steps this workflow performs?
1. **Checkout code** – retrieves the repository's code onto the runner.
2. **Validate HTML** – runs an HTML5 validator against the site files.
3. **Check links** – scans for broken markdown links.
4. **Upload artifact** – packages the site files and uploads them as a Pages deployment artifact.

## What does the "Checkout code" step do and why is it necessary?
It uses `actions/checkout@v4` to clone the repository's code onto the GitHub Actions runner. It's necessary because runners start with an empty environment, so the workflow needs the repository files present locally before it can validate HTML, check links, or upload anything.

## What is the purpose of the environment configuration?
The `deploy` job defines an `environment: { name: github-pages, url: ... }`. This associates the job with GitHub's `github-pages` deployment environment, which enables the required `pages`/`id-token` permissions, tracks deployment history, and exposes the live site URL (via `steps.deployment.outputs.page_url`) in the deployment summary.

## How does this automated deployment improve reliability compared to manual deployment?
- Every push/PR is validated (HTML validation, link checking) before anything is deployed, catching errors automatically instead of relying on someone remembering to check.
- Deployment only happens after the build-and-test job succeeds (`needs: build-and-test`), preventing broken code from reaching production.
- The process is consistent and repeatable every time, removing human error from manual copy/upload steps.
- The `if` condition restricts actual deployment to pushes on `main`, avoiding accidental deploys from other branches or PRs.

## What would happen if you pushed code to a different branch (not main)?
The `push` trigger only matches `main`, so pushing to another branch would not run the workflow at all (unless that push is part of a pull request targeting `main`, in which case the `build-and-test` job would run for validation, but the `deploy` job would be skipped since its `if` condition requires `github.ref == 'refs/heads/main'`).
