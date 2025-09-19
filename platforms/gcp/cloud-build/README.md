# Next.js + Bugster (via GCP Cloud Build)

This is a [Next.js](https://nextjs.org) project bootstrapped with [`create-next-app`](https://nextjs.org/docs/app/api-reference/cli/create-next-app).

It also includes a **Cloud Build integration with Bugster** so that every **Pull Request targeting `main`** will automatically notify Bugster and trigger end-to-end tests.

---

## 🚀 Getting Started (Next.js)

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

## Trigger Bugster with Cloud Build

This project is configured to call Bugster automatically on every Pull Request to main using Google Cloud Build.

1. Cloud Build Config:

Inside this repo, you’ll find a file at:

/platforms/gcp/cloud-build/cloudbuild.yaml


This file defines a simple curl step that POSTs to Bugster’s Custom Integration Webhook:
```bash
steps:
  - name: 'curlimages/curl'
    entrypoint: 'curl'
    args:
      - '-X'
      - 'POST'
      - 'https://api.bugster.app/webhooks/integrations/custom'
      - '-H'
      - 'Content-Type: application/json'
      - '-H'
      - 'X-API-KEY: ${_BUGSTER_API_KEY}'
      - '--data'
      - |
        {
          "deployment_state": "success",
          "project_id": "${_PROJECT_ID}",
          "organization_id": "${_ORG_ID}",
          "branch": "${_BRANCH_NAME}",
          "environment_url": "${_DEPLOYMENT_URL}",
          "environment": "production"
        }
options:
  logging: CLOUD_LOGGING_ONLY
```

2. Substitution Variables

The following substitutions must be configured in your Cloud Build Trigger:

Variable	Example Value	Notes
_BUGSTER_API_KEY	bugster_xxx	Bugster API Key (from dashboard)
_PROJECT_ID	0910-1757542431	Bugster Project ID
_ORG_ID	org_30bL1wM4DbhHnU85252QKw5	Bugster Organization ID
_DEPLOYMENT_URL	https://preview.example.com	Public URL of your PR preview/deployment
_BRANCH_NAME	main	Target branch (fixed to main)

_BUGSTER_API_KEY, _PROJECT_ID and _ORG_ID can be found in your project Settings under integrations tab.
 
Then, these variables are set once in Cloud Build → Trigger → Substitution variables.

3. Create the Trigger in GCP

Go to Cloud Build → Triggers → Create Trigger.

Select your GitHub repository.

Event: Pull request → Target branch = main.

Configuration file: /platforms/gcp/cloud-build/cloudbuild.yaml.

Add the substitution variables listed above.

Save.

4. Flow

Developer opens a PR to main.

Cloud Build runs the trigger.

The curl step sends a webhook to Bugster.

Bugster receives the deployment URL and executes tests automatically.