# URLWatch Website Change Detector GitHub Action

A GitHub Action that monitors websites for changes using [urlwatch](https://thp.io/2008/urlwatch/). Get notified through GitHub Issues when your monitored pages change, with full history tracking and configurable filters.

## Features

🌐 **Monitor any web resource** - Track HTML pages, JSON APIs, XML feeds, or any URL-accessible content  
🔍 **Smart filtering** - Use CSS selectors, XPath, JSON paths, or regex to watch specific content  
📦 **State management** - Maintains change history between runs using GitHub Artifacts  
📬 **GitHub integration** - Automatic issue creation for detected changes  
🛠 **Extensible** - Add custom filters and processing logic through Python hooks  
📊 **Dual outputs** - Get raw changes and formatted markdown reports  

## Quick Start

1. **Create configuration directory** in your repo:
   ```bash
   mkdir -p .github/urlwatch
   ```

2. **Add monitoring targets** in `.github/urlwatch/urls.yml`:
   ```yaml
   - name: Example Documentation
     url: https://docs.example.com
     filter:
       - css:div.main-content
       - html2text
   ```

3. **Create workflow** (`.github/workflows/monitor.yml`):
   ```yaml
   name: Website Change Monitoring
   on:
     schedule:
       - cron: '0 */6 * * *'  # Every 6 hours
     workflow_dispatch:
   
   permissions:
     actions: read
     contents: write
     issues: write
   
   jobs:
     monitor:
       runs-on: ubuntu-latest
       steps:
         - uses: nilp0inter/urlwatch-action@v1
           with:
             config-dir: '.github/urlwatch'
             cache-name: 'website-monitor-state'
   ```

## Configuration

### Action Inputs

| Parameter | Description | Default |
|-----------|-------------|---------|
| `config-dir` | Path to urlwatch config files | `.github/urlwatch` |
| `cache-name` | Artifact name for storing state | `urlwatch-cache` |
| `create-issues` | Create GitHub Issues for changes | `true` |
| `issue-header` | Issue title template | `Website changes detected 🌐` |
| `issue-tags` | Comma-separated issue labels | `monitoring,update` |

### Required Files

1. **urls.yml** - Main configuration file:
   ```yaml
   - name: "API Status"
     url: "https://api.example.com/status"
     filter:
       - json:status
       - strip
   ```

2. **hooks.py** (optional) - Custom filters/processors:
   ```python
   def filter_price(value):
       return float(value.replace('$', ''))
   ```

3. **config.yml** (optional) - urlwatch settings:
   ```yaml
   report:
     html: true
     diff: unified
   ```

## Outputs

Access monitoring results in subsequent steps:
```yaml
- name: Show results
  run: |
    echo "${{ steps.monitor.outputs.changes }}"
    echo "${{ steps.monitor.outputs.report }}"
```

| Output | Description |
|--------|-------------|
| `changes` | Raw text output from urlwatch |
| `report` | Markdown-formatted change report |

## Example Use Cases

- **Documentation tracking**: Get notified when API docs change
- **Status monitoring**: Watch for service status page updates
- **Price tracking**: Monitor e-commerce product pages
- **Competitor analysis**: Track changes to competitor websites
- **Regulatory compliance**: Watch for legal document updates

## Advanced Usage

### Custom Filters
```python
# hooks.py
from urlwatch.filters import FilterBase

class EmojiCounter(FilterBase):
    def filter(self, data):
        return f"😀 Count: {data.count('😀')}"
```

### Workflow Integration
```yaml
- name: Post to Slack
  if: ${{ steps.monitor.outputs.changes != '' }}
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "${{ steps.monitor.outputs.report }}"
      }
```

## Contributing

Contributions welcome! Please follow these steps:
1. Fork the repository
2. Create a feature branch
3. Submit a Pull Request

## License

MIT License - See [LICENSE](LICENSE) for details

---

**Maintained by** Roberto Abdelkader Martínez Pérez | **Credits** to [urlwatch](https://thp.io/2008/urlwatch/) authors
