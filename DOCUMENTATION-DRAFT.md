# GitHub Actions Workflow: Sync Postman Collection

## Overview

The `sync-postman-collection.yml` workflow automates the process of converting OpenAPI specifications into Postman collections and synchronizing them to a Postman workspace. This enables API documentation and testing to stay in sync with specification changes with **minimal developer effort.**

## Workflow Purpose

This workflow performs the following tasks:

1. **Checkout Repository** — Retrieves the latest code from the repository
2. **Setup Node.js** — Prepares the Node.js runtime environment
3. **Install openapi-to-postman** — Installs the CLI tool for OpenAPI→Postman conversion
4. **Build Postman Collection** — Converts an OpenAPI spec file into a Postman collection JSON file
5. **Sync to Postman** — Uploads or updates the collection in your Postman workspace via API

## Workflow Triggers

The workflow runs automatically in two scenarios:

- **On Manual Dispatch** — Triggered via GitHub Actions UI using `workflow_dispatch`
- **On Spec Changes** — Automatically runs when any file in the `specs/` folder is modified or when the workflow file itself changes

## Configuration Variables

The workflow uses environment variables in the `env` section to control its behavior:

| Variable | Purpose | Example |
|----------|---------|---------|
| `COLLECTION_NAME` | Name of the collection in Postman | `'Claims Processing API'` |
| `TARGET_SPEC` | OpenAPI spec filename in `specs/` folder | `'claims-processing-api-openapi.yaml'` |
| `NODE_VERSION` | Node.js version to use | `'20'` |

## Secrets Required

The workflow requires two GitHub repository secrets:

- **`POSTMAN_API_KEY`** — Your Postman API key (found in Postman settings)
- **`POSTMAN_ACCESS_TOKEN`** — Your Postman personal access token (found in Postman settings)

### Setting Up Secrets

1. Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Add `POSTMAN_API_KEY` with your Postman API key
4. Add `POSTMAN_ACCESS_TOKEN` with your Postman personal access token

## How It Works

### Step-by-Step Execution

1. **Checkout** — Code is pulled from the repository
2. **Node.js Setup** — Environment is configured for running npm packages
3. **Tool Installation** — `openapi-to-postmanv2` CLI is installed globally
4. **Collection Generation** — The OpenAPI spec is converted to `collection.json` using:
   ```
   openapi2postmanv2 -s specs/<TARGET_SPEC> -o collection.json -p -O folderStrategy=Tags,includeAuthInfoInExample=false
   ```
5. **Postman Sync** — The workflow queries the Postman API to:
   - Search for an existing collection matching `COLLECTION_NAME`
   - **If found** — Updates the existing collection via PUT request
   - **If not found** — Creates a new collection via POST request

### API Sync Logic

The sync script uses the Postman API to:
- Query all collections in your workspace
- Match by collection name
- Update via `PUT /collections/{uid}` if collection exists
- Create via `POST /collections` if collection is new

## Adapting the Workflow to a New API Spec

To use this workflow for a different OpenAPI spec:

### Step 1: Add Your OpenAPI Spec
Place your OpenAPI specification file in the `specs/` folder:
```
specs/your-api-openapi.yaml
```

### Step 2: Update Workflow Configuration

Edit `.github/workflows/sync-postman-collection.yml` and modify the `env` section:

```yaml
env:
  COLLECTION_NAME: 'Your API Name'          # Name for this collection in Postman
  TARGET_SPEC: "your-api-openapi.yaml"      # Filename in specs/ folder
  NODE_VERSION: '20'
```

**Example for Payment Refund API:**
```yaml
env:
  COLLECTION_NAME: 'Payment Refund API'
  TARGET_SPEC: "payment-refund-api-openapi.yaml"
  NODE_VERSION: '20'
```

### Step 3: Trigger the Workflow

Choose one of:
- **Manual Trigger** — Go to **Actions** → **Sync Postman Collection** → **Run workflow**
- **Automatic Trigger** — Commit any change to `specs/your-api-openapi.yaml`

### Step 4: Verify in Postman

After the workflow completes:
1. Log into your Postman workspace
2. Look for a collection named with your `COLLECTION_NAME` value
3. Verify requests are properly organized and documented

## Customizing OpenAPI Conversion Options

The workflow passes conversion options to `openapi2postmanv2`. Current options:

```
folderStrategy=Tags           # Organize requests by OpenAPI tags
includeAuthInfoInExample=false # Don't include auth in example requests
```

To customize these:
- Edit the conversion command in the **"Build Postman collection from OpenAPI"** step
- See `openapi-to-postmanv2` documentation for available options

## Troubleshooting

### Workflow Fails to Run

**Check:**
- GitHub secrets are set (`POSTMAN_API_KEY`, `POSTMAN_ACCESS_TOKEN`)
- OpenAPI spec file exists in `specs/` folder with correct filename
- Filename in `TARGET_SPEC` matches exactly (case-sensitive)

### Collection Not Appearing in Postman

**Check:**
- Workflow completed successfully (check **Actions** tab for logs)
- `COLLECTION_NAME` matches what appears in Postman
- Postman access token has appropriate permissions

### Collection Update Fails

**Common Causes:**
- Malformed OpenAPI specification (run local validation)
- Token permissions insufficient for updates
- Collection name mismatch between GitHub and Postman

**Debug:**
- Check workflow logs in **Actions** tab
- Manually test Postman API endpoints using `curl` with your token

## Multiple Collections in One Repository

To manage multiple APIs in the same repository, you have two options:

### Option 1: Separate Workflows
Create individual workflow files (e.g., `sync-claims-api.yml`, `sync-payment-api.yml`) with different configurations.

### Option 2: Matrix Strategy
Extend the workflow to use a GitHub Actions matrix:
```yaml
strategy:
  matrix:
    api:
      - { name: 'Claims Processing API', spec: 'claims-processing-api-openapi.yaml' }
      - { name: 'Payment Refund API', spec: 'payment-refund-api-openapi.yaml' }
```

Then reference matrix variables in steps: `${{ matrix.api.name }}`, `${{ matrix.api.spec }}`

## Files Involved

| File | Purpose |
|------|---------|
| `.github/workflows/sync-postman-collection.yml` | Main workflow definition |
| `specs/*.yaml` | OpenAPI specification files |
| `collection.json` | Generated Postman collection (local artifact) |

## Related Resources

- [OpenAPI to Postman Converter Docs](https://github.com/postmanlabs/openapi-to-postman)
- [Postman API Documentation](https://www.postman.com/postman/workspace/postman-public/documentation/12959542-c8142d51-e97c-46b5-8c55-fdbaddc26673)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
