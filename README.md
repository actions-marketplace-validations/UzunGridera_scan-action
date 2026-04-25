# AgentMinds Scan — GitHub Action

Run an [AgentMinds](https://agentminds.dev) site scan in CI. Posts a PR comment with grade, top issues, and a link to the full report. No signup, no API key.

## Quick start

```yaml
name: Site quality

on:
  pull_request:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: agentminds/scan-action@v1
        with:
          url: https://your-preview-deployment.example.com
          fail-on-grade: D   # fail the workflow if grade is D or worse
```

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `url` | Yes | — | URL to scan. Typically a preview deployment URL. |
| `fail-on-grade` | No | `F` | Fail the job if scan grade ≤ this. Use `F` to only fail on outright failure, `D` to enforce passing grade, etc. |
| `comment` | No | `true` | Post a sticky PR comment with the scan summary. |
| `github-token` | No | `${{ github.token }}` | Token for posting PR comments. Default usually fine. |

## Outputs

| Name | Description |
|---|---|
| `grade` | A / B / C / D / F |
| `score` | 0-100 |
| `scan-url` | Shareable link to the full report on agentminds.dev |

## What gets checked

Every scan runs 50+ checks across:

- **Security headers** (HSTS, CSP, X-Frame-Options, COOP, etc.)
- **SEO** (title/meta tags, canonical, OG, sitemap, robots.txt)
- **AEO** (llms.txt, structured data, FAQPage, AI bot blocking)
- **Performance** (latency, content size, redirect count, mixed content)
- **Accessibility** (alt text, lang attribute, focus indicators)

## PR comment example

> ## ![AgentMinds Grade](badge.svg) AgentMinds scan: `https://preview.example.com`
>
> | Grade | Overall | Security | SEO/AEO | Latency |
> |---|---|---|---|---|
> | **B** | 78/100 | 60/100 | 90/100 | 412ms |
>
> ### Top issues
> - 🛡 **critical** Content-Security-Policy header missing
> - 🔍 **warning** Description length 88 (optimal: 150-160)
>
> [Full report →](https://agentminds.dev/scan/abc123)

The comment is sticky — re-runs update the existing comment instead of spamming.

## Recipes

### Run on every PR, but never block

```yaml
- uses: agentminds/scan-action@v1
  with:
    url: ${{ steps.deploy.outputs.preview_url }}
    fail-on-grade: F
```

### Block PRs that drop below grade C

```yaml
- uses: agentminds/scan-action@v1
  with:
    url: ${{ steps.deploy.outputs.preview_url }}
    fail-on-grade: C
```

### Use the outputs

```yaml
- uses: agentminds/scan-action@v1
  id: scan
  with:
    url: ${{ steps.deploy.outputs.preview_url }}

- run: echo "Site graded ${{ steps.scan.outputs.grade }} - report at ${{ steps.scan.outputs.scan-url }}"
```

### Comment-only mode (don't fail on bad grade)

```yaml
- uses: agentminds/scan-action@v1
  with:
    url: ${{ steps.deploy.outputs.preview_url }}
    fail-on-grade: F   # never fail
    comment: true
```

## Privacy

The action calls the public [free-scan API](https://api.agentminds.dev/api/v1/free-scan). It only sends the URL you provide. Scan results are stored on AgentMinds and are accessible at the returned `share_url` — these are public by design (the URL acts as the only access token, akin to how Lighthouse PageSpeed Insights URLs work).

## License

MIT.
