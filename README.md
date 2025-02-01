# URLWatch Website Change Detector GitHub Action 🔍

Automated website change detection using GitHub Actions and [urlwatch](https://thp.io/2008/urlwatch/). Monitor web pages, APIs, or documents for modifications and get automatic GitHub issues with detailed diffs.

**Key Features**:
- 🕵️♂️ Detect content changes with CSS/XPath filters
- 📬 Automatic issue creation with configurable labels
- 💾 State preservation between runs
- ⚙️ Full urlwatch configuration support
- 🛡️ Security-focused design (no credential storage)

## Quick Start 🚀

### Basic Configuration

1. Create config directory:
   ```bash
   mkdir -p .github/urlwatch
   ```

2. Add monitoring targets (`.github/urlwatch/urls.yml`):
   ```yaml
   - name: Example Privacy Policy
     url: https://example.com/privacy
     filter:
       - css: article.main-content
   ```

3. Create workflow (`.github/workflows/monitor.yml`):
   ```yaml
   name: Website Monitoring
   on:
     schedule:
       - cron: '0 8 * * *'  # Daily at 8 AM

   jobs:
     monitor:
       runs-on: ubuntu-latest
       steps:
         - uses: nilp0inter/urlwatch-action@v1
           with:
             token: ${{ secrets.GITHUB_TOKEN }}
   ```

## Configuration ⚙️

### Required Files

| File | Purpose | Documentation |
|------|---------|---------------|
| `urls.yml` | List of URLs to monitor | [URLs Configuration](https://urlwatch.readthedocs.io/en/latest/jobs.html) |
| `urlwatch.yaml` | Monitoring preferences | [Settings Guide](https://urlwatch.readthedocs.io/en/latest/configuration.html) |

### Action Inputs

| Parameter | Description | Default |
|-----------|-------------|---------|
| `config-dir` | Path to urlwatch config | `.github/urlwatch` |
| `cache-name` | Cache artifact name | `urlwatch-cache` |
| `create-issues` | Create issues on changes | `true` |
| `issue-header` | Issue title template | `Website changes detected 🌐` |
| `issue-tags` | Comma-separated labels | `monitoring,update` |
| `token` | GitHub token | **Required** |

### Security Notice 🔒

1. **Never store credentials** in configuration files
2. This action only uses the **text reporter** for GitHub issues
3. Sensitive filters should be added via `hooks.py` ([Filter Documentation](https://urlwatch.readthedocs.io/en/latest/filters.html))

## Advanced Usage 💡

### Custom Workflow Example

```yaml
- uses: nilp0inter/urlwatch-action@v1
  with:
    config-dir: 'config/monitoring'
    cache-name: 'custom-cache'
    issue-header: '[Alert] Change detected: ${{ now }}'
    issue-tags: 'urgent,legal'
    token: ${{ secrets.GITHUB_TOKEN }}
```

### Integration with Other Actions

```yaml
- name: Run Monitor
  uses: nilp0inter/urlwatch-action@v1
  id: urlwatch
  token: ${{ secrets.GITHUB_TOKEN }}

- name: Post to Slack
  if: ${{ steps.urlwatch.outputs.changes != '' }}
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Detected changes: ${{ steps.urlwatch.outputs.report }}"
      }
```


## Documentation Links 📚

- [Official urlwatch Documentation](https://urlwatch.readthedocs.io/)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows)
