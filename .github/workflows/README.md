# GitHub Actions Workflows

## API Testing and Validation Workflow

This workflow automatically validates your API specification and tests your API endpoints on every pull request and merge to main.

### What it does

1. **Lint OpenAPI Specification** - Validates the OpenAPI spec using Spectral
2. **Run Postman Collection** - Executes all API tests using Postman CLI

### Setup Instructions

#### 1. Create a Postman API Key (Optional)

The workflow can run without authentication, but if you want to use cloud features:

1. Go to [Postman API Keys](https://go.postman.co/settings/me/api-keys)
2. Click "Generate API Key"
3. Give it a name (e.g., "GitHub Actions")
4. Copy the generated key (starts with `PMAK-`)

#### 2. Add GitHub Secret

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `POSTMAN_API_KEY`
5. Value: Paste your Postman API key
6. Click **Add secret**

> **Note:** The workflow will still run without this secret, but you won't be able to use Postman cloud features.

### Workflow Triggers

The workflow runs automatically on:
- **Pull Requests** to `main` branch
- **Pushes** to `main` branch (merges)

### Workflow Jobs

#### Job 1: Lint OpenAPI Spec
- Uses Spectral CLI to validate OpenAPI specification
- Checks for:
  - Valid OpenAPI 3.0 syntax
  - Best practices compliance
  - Common API design issues
- Fails the build if critical issues are found

#### Job 2: Run Postman Collection Tests
- Starts the Identity Service locally
- Installs Postman CLI
- Runs the complete collection with all tests
- Generates test reports (JSON format)
- Uploads test results as artifacts

### Viewing Test Results

1. Go to the **Actions** tab in your GitHub repository
2. Click on the workflow run
3. View the job logs for detailed output
4. Download test results from the **Artifacts** section

### Test Reports

Test results are saved as JSON and uploaded as artifacts:
- **Artifact name:** `postman-test-results`
- **Retention:** 30 days
- **Format:** JSON (Postman collection run format)

### Customization

#### Change Node.js Version
Edit the `node-version` in both jobs:
```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'  # Change this
```

#### Add More Reporters
Modify the Postman CLI command to add reporters:
```yaml
postman collection run "postman/collections/Identity Service API.postman_collection.json" \
  --reporters cli,json,junit,html \
  --reporter-json-export test-results.json \
  --reporter-junit-export junit-results.xml \
  --reporter-html-export test-report.html
```

#### Change Spectral Rules
Create a custom `.spectral.yaml` file in your repository root:
```yaml
extends: spectral:oas
rules:
  # Add custom rules here
```

Then update the lint command:
```yaml
- name: Lint OpenAPI Spec
  run: spectral lint postman/specs/openapi.yaml
```

### Troubleshooting

#### Server doesn't start
- Check that `npm start` command is defined in `package.json`
- Verify port 3001 is not in use
- Check server logs in the workflow output

#### Postman CLI login fails
- Verify `POSTMAN_API_KEY` secret is set correctly
- Check that the API key is valid and not expired
- The workflow continues even if login fails (for local collection runs)

#### Tests fail
- Check the test results artifact for detailed error messages
- Verify the API is responding correctly
- Review the Postman collection test scripts

### Local Testing

Test the workflow locally before pushing:

```bash
# Lint the OpenAPI spec
npm install -g @stoplight/spectral-cli
spectral lint postman/specs/openapi.yaml --ruleset spectral:oas

# Run the collection with Postman CLI
npm start &
postman collection run "postman/collections/Identity Service API.postman_collection.json" \
  --env-var "baseUrl=http://localhost:3001"
```

### CI/CD Integration

This workflow integrates seamlessly with:
- **Branch protection rules** - Require checks to pass before merging
- **Status checks** - See test results directly in PRs
- **Automated deployments** - Trigger deployments after successful tests

### Best Practices

1. **Always run tests locally** before pushing
2. **Keep API key secure** - Never commit it to the repository
3. **Review test failures** in the Actions tab
4. **Update tests** when API changes
5. **Monitor test execution time** and optimize if needed
