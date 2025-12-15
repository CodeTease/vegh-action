# Vegh Action 🥬

The official GitHub Action for **Vegh** - The snapshot tool by **CodeTease**.

This action is powered by the **PyVegh** engine, bringing its exclusive features (such as **Vegh Hooks** and the colorful **LOC Analytics Dashboard**) directly into your CI/CD pipeline. It maintains full compatibility with the core Vegh format.

> "Tight packing, swift unpacking, no nonsense."

## Usage

1. **Analytics & PR Comment**

Count LOC and impress your reviewers with a beautiful report.
```yaml
name: Analytics
on: [pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Use vegh-action
      - name: Vegh Analysis
        id: vegh
        uses: codetease/vegh-action@v1
        with:
          command: 'loc'
          path: './src'

      # Automatically comment on the PR using the LOC report output
      - name: Comment PR
        uses: actions/github-script@v6
        with:
          script: |
            const report = `${{ steps.vegh.outputs.loc-report }}`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `### 📊 CodeTease Analytics\n\n\`\`\`\n${report}\n\`\`\``
            })
```

2. **Manual Snapshot & Transfer**

Total autonomy! Snap your build artifacts, then send them (or not).
```yaml
name: Release
on: [push]

jobs:
  build-and-ship:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # 1. Build your app
      - name: Build
        run: npm install && npm run build
        
      # 2. Create Snapshot (Note: Vegh Hooks from .veghhooks.json will run here)
      - name: Pack Artifact
        uses: codetease/vegh-action@v1
        with:
          command: 'snap'
          path: './dist'
          output: 'release.vegh'
          comment: 'Release build ${{ github.sha }}'
          
      # 3. Send to Server (Optional)
      - name: Upload to Teaserverse
        uses: codetease/vegh-action@v1
        with:
          command: 'send'
          path: 'release.vegh'
          url: 'https://cdn.teaserverse.online/upload'
          token: ${{ secrets.VEGH_TOKEN }}
```

## Inputs

Inputs can be passed using the `with:` context in your workflow step.
* `command` (Required): The Vegh command to run. Options: `snap`, `loc`, `send`, `restore`. (Default: `snap`)
* `path` (Required): The source path. (Folder for `snap`/`loc`, `.vegh` file for `send`/`restore`). (Default: `.`)
* `output` (Optional): Output filename for `snap` or destination folder for `restore`. (Default: `artifact.vegh`)
* `token` (Optional): Bearer authentication token for the `send` command.
* `url` (Optional): Target upload URL for the `send` command.
* `comment` (Optional): Metadata comment for the snapshot (`snap`). (Default: `Snapshot via GitHub Actions`)
* `dry-run` (Optional): Set to `true` to simulate the snapshot process (`snap`). (Default: `false`)
* `python-version` (Optional): Python version to use for setup. (Default: `3.12`)

# License

[MIT](LICENSE)
