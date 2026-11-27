# Web Hosting — S-03

## Decision

- Provider: GitHub Pages
- Plan: Free
- Repository visibility: Public
- Deployment method: GitHub Actions
- Production branch: main
- Publish directory: web/
- HTTPS: Enabled
- Expected URL: https://ak-ta-labs.github.io/leonida-daily/

## Rationale

- Free static website hosting.
- Native GitHub integration.
- Compatible with HTML, CSS and JavaScript.
- Automatic deployment after merging changes into main.
- No additional hosting account required.

## Deployment

Workflow: .github/workflows/deploy.yml

The workflow temporarily copies designHTML.html to index.html
if the latter does not exist.

## Security

- No credentials or API keys in frontend files.
- Production deployments are restricted to main.
- Branch protection requires one approving review.

## Pending validation

- First successful production deployment.
- Website functionality after deployment.
- Pull request preview deployment strategy.
