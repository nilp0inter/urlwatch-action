# URLWatch Website Change Detector GitHub Action 🔍

Automated website change detection using GitHub Actions and [urlwatch](https://thp.io/2008/urlwatch/). Monitor web pages, APIs, or documents for modifications and get structured outputs for integration with other actions.

**Key Features**:
- 🕵️♂️ Direct configuration through workflow inputs
- 📤 Structured outputs for detected changes
- 💾 Automatic cache preservation between runs
- ⚙️ Full urlwatch configuration flexibility
- 🛡️ Secure secret handling for sensitive filters

## Quick Start 🚀

### Basic Configuration

1. Create workflow (`.github/workflows/monitor.yml`):
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
             urls: |
               - name: Example Privacy Policy
                 url: https://example.com/privacy
                 filter:
                   - css: article.main-content
             token: ${{ secrets.GITHUB_TOKEN }}
   ```

## Configuration ⚙️

### Action Inputs

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `urls` | YAML content for urlwatch targets | Yes | - |
| `settings` | YAML content for urlwatch config | No | [Default settings](#default-settings) |
| `hooks` | Python code for advanced filters | No | - |
| `cache-name` | Cache artifact name | No | `urlwatch-cache` |
| `token` | GitHub token | Yes | - |

### Default Settings
```yaml
display:
  empty-diff: true
  error: true
  new: true
  unchanged: false
job_defaults:
  all:
    treat_new_as_changed: true
  browser: {}
  shell: {}
  url: {}
report:
  stdout:
    color: false
    enabled: true
  text:
    details: true
    footer: false
    line_length: 120
    minimal: false
    separate: false
```

## Output Handling 📤

The action provides two outputs for downstream processing:

| Output | Description | Format Example |
|--------|-------------|----------------|
| `changes` | Raw detection output | Plain text |
| `report` | Formatted report | Markdown codeblock |

## Advanced Usage 💡

### Custom Configuration Example

```yaml
- uses: nilp0inter/urlwatch-action@v1
  with:
    urls: |
      - name: Secure Document Check
        url: https://api.example.com/v1/document
        filter:
          - xpath: //div[@id="content"]
    settings: |
      job_defaults:
        url:
          headers:
            Authorization: ${{ secrets.API_TOKEN }}
    hooks: |
      def filter_secure_content(value, *args, **kwargs):
          return value.replace(${{ secrets.SENSITIVE_PATTERN }}, "REDACTED")
    token: ${{ secrets.GITHUB_TOKEN }}
```

### Change Notification Workflow

```yaml
- name: Run Monitor
  uses: nilp0inter/urlwatch-action@v1
  id: urlwatch
  with:
    urls: |
      - url: https://example.com/tos
        filter: line
    token: ${{ secrets.GITHUB_TOKEN }}

- name: Create GitHub Issue
  if: ${{ steps.urlwatch.outputs.changes != '' }}
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.create({
        owner: context.repo.owner,
        repo: context.repo.repo,
        title: 'Website Content Update',
        body: 'Detected changes:\n\n' + '${{ steps.urlwatch.outputs.report }}',
        labels: ['monitoring']
      })
```

- [urlwatch Official Documentation](https://urlwatch.readthedocs.io/)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows)
