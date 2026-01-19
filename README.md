# Squidler GitHub Action

Trigger Squidler quality checks from your GitHub Actions workflows. This action integrates Squidler's automated testing into your CI/CD pipeline, running test cases and reporting results directly in GitHub check runs.

## Features

- 🚀 **Trigger tests** on every push, pull request, or deployment
- ✅ **GitHub Check Runs** - Results appear natively in GitHub's UI
- 🏷️ **Label-based test selection** - Control which tests run via Squidler labels
- 🔒 **Secure** - Uses Squidler Site API keys for authentication
- 📊 **Comprehensive error handling** - Clear feedback on configuration issues

## Prerequisites

Before using this action, you need to:

1. **Set up Squidler GitHub Integration**
   - Install the Squidler GitHub App on your organization
   - Link your GitHub repository to a Squidler site
   - Configure labels for your repository in Squidler
   - Assign labels to test cases you want to run

2. **Get your Site API Key**
   - Go to your site settings in Squidler
   - Generate a Site API Key
   - Add it as a GitHub secret (`SQUIDLER_API_KEY`)

## Usage

See the [examples/](examples/) directory for complete workflow examples.

### Basic Example

```yaml
name: Squidler Quality Checks

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  squidler-checks:
    runs-on: ubuntu-latest
    steps:
      - name: Run Squidler Quality Checks
        uses: squidlerio/actions@v1
        with:
          api-key: ${{ secrets.SQUIDLER_API_KEY }}
          repository-id: ${{ github.repository_id }}
          sha: ${{ github.sha }}
          ref: ${{ github.ref }}
```

### Advanced Example - Wait for Deployment

If you deploy before testing, wait for the deployment to complete:

```yaml
name: Deploy and Test

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to production
        run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}

      # Wait for deployment to be live
      - name: Wait for deployment
        run: |
          echo "Waiting for deployment to be available..."
          for i in {1..30}; do
            if curl -f -s https://yoursite.com/health > /dev/null; then
              echo "Deployment is live!"
              exit 0
            fi
            echo "Waiting... ($i/30)"
            sleep 10
          done
          echo "Deployment health check timeout"
          exit 1

  test:
    runs-on: ubuntu-latest
    needs: deploy
    steps:
      - name: Run Squidler Quality Checks
        uses: squidlerio/actions@v1
        with:
          api-key: ${{ secrets.SQUIDLER_API_KEY }}
          repository-id: ${{ github.repository_id }}
          sha: ${{ github.sha }}
          ref: ${{ github.ref }}
```

### Custom API URL (for testing/enterprise)

```yaml
- name: Run Squidler Quality Checks
  uses: squidlerio/actions@v1
  with:
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
    api-url: 'https://api.dev.squidler.io'  # Optional: defaults to production
    repository-id: ${{ github.repository_id }}
    sha: ${{ github.sha }}
    ref: ${{ github.ref }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `api-key` | Your Squidler Site API Key | Yes | - |
| `repository-id` | GitHub repository ID | Yes | - |
| `sha` | Commit SHA | Yes | - |
| `ref` | Branch reference | Yes | - |
| `api-url` | Squidler API URL | No | `https://api.squidler.io` |

## Outputs

| Output | Description |
|--------|-------------|
| `status` | Status of the test trigger (`success` or `failure`) |
| `message` | Response message from Squidler API |

### Using Outputs

```yaml
- name: Run Squidler Quality Checks
  id: squidler
  uses: squidlerio/actions@v1
  with:
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
    repository-id: ${{ github.repository_id }}
    sha: ${{ github.sha }}
    ref: ${{ github.ref }}

- name: Check result
  run: |
    echo "Status: ${{ steps.squidler.outputs.status }}"
    echo "Message: ${{ steps.squidler.outputs.message }}"
```

## How It Works

1. **Action triggers** → Sends request to Squidler API with repository ID, commit SHA, and branch
2. **API validates** → Checks repository is linked, labels configured, test cases exist
3. **Tests queued** → Test cases with matching labels are queued for execution
4. **Check runs created** → GitHub check runs appear in the UI for each test case
5. **Tests execute** → Squidler runs the tests against your live site
6. **Results reported** → Check runs update with pass/fail status and details

## Viewing Results

After the action runs successfully, check runs will appear in:

- **Pull Request Checks** - In the "Checks" tab of your PR
- **Commit Status** - On the commit page in GitHub
- **GitHub API** - Via the Checks API

Each test case creates a separate check run with:
- ✅ **Pass/Fail status**
- 📝 **Detailed results** and screenshots
- 🔗 **Link to Squidler** for full test details

## Troubleshooting

### "Repository not linked to any site"

The GitHub repository needs to be connected to a Squidler site:
1. Go to Squidler and navigate to your site settings
2. Connect your GitHub repository under "GitHub Integration"

### "No labels configured for this repository"

You need to assign labels to this repository:
1. In Squidler, go to your site settings
2. Select your GitHub repository
3. Add one or more labels to specify which test cases should run

### "No test cases found with the configured labels"

No test cases match the labels you've assigned to this repository:
1. Verify which labels are assigned to your repository in Squidler
2. Add those labels to your test cases, or update the repository labels

### "Invalid API key or insufficient permissions"

The API key is invalid or has been revoked:
1. Generate a new Site API Key in Squidler
2. Update your GitHub secret `SQUIDLER_API_KEY`

## Security

- ✅ **Never commit your API key** - Always use GitHub Secrets
- ✅ **Limit secret access** - Use environment-specific secrets if needed
- ✅ **Rotate keys** - Regenerate API keys periodically

## Support

- 📖 **Documentation**: [https://squidler.io/docs](https://squidler.io/docs)
- 💬 **Support**: support@squidler.io
- 🐛 **Issues**: [GitHub Issues](https://github.com/squidlerio/actions/issues)

## License

MIT License - see [LICENSE](LICENSE) file for details
