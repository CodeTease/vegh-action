# Vegh Action 🥬

The official GitHub Action for **Vegh** - The snapshot tool by **CodeTease**.

This action is powered by the **PyVegh** engine, bringing its exclusive features (such as **Vegh Hooks** and the colorful **LOC Analytics Dashboard**) directly into your CI/CD pipeline. It maintains full compatibility with the core Vegh format.

> "Tight packing, swift unpacking, no nonsense."

## Usage

### 1. Analytics & PR Comment

Count LOC and impress your reviewers with a beautiful report.

```yaml
name: Analytics
on: [pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Use vegh-action (Standard LOC)
      - name: Vegh Analysis
        id: vegh
        uses: codetease/vegh-action@v1
        with:
          command: 'loc'
          path: './src'

      # OR Use SLOC (Source Lines of Code - excludes comments/blanks)
      - name: Vegh SLOC Analysis
        id: vegh-sloc
        uses: codetease/vegh-action@v1
        with:
          command: 'loc'
          path: './src'
          sloc: 'true'

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

### 2. Manual Snapshot & Transfer

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

      # 2. Security Audit (New!)
      # Scans for secrets and suspicious files before packing
      - name: Security Audit
        uses: codetease/vegh-action@v1
        with:
          command: 'audit'
          path: '.'
        
      # 3. Create Snapshot (Note: Vegh Hooks from .veghhooks.json will run here)
      - name: Pack Artifact
        uses: codetease/vegh-action@v1
        with:
          command: 'snap'
          path: './dist'
          output: 'release.vegh'
          comment: 'Release build ${{ github.sha }}'
          
      # 4. Send to Server (Optional)
      - name: Upload to Teaserverse
        uses: codetease/vegh-action@v1
        with:
          command: 'send'
          path: 'release.vegh'
          url: 'https://cdn.teaserverse.online/upload'
          token: ${{ secrets.VEGH_TOKEN }}
```

### 3. AI Context Generation (New!)

Generate a clean XML context file for AI workflows.

```yaml
steps:
  - name: Generate AI Context
    id: context
    uses: codetease/vegh-action@v1
    with:
      command: 'prompt'
      path: './src'
      clean: 'true' # Removes locks, secrets, and temp files
      
  - name: Use Context
    run: echo "Context file is at ${{ steps.context.outputs.context-path }}"
```

### 4. Integrity Check

Verify the integrity of a restored artifact or cache using Blake3.

```yaml
steps:
  - name: Verify Artifact
    uses: codetease/vegh-action@v1
    with:
      command: 'check'
      path: 'artifact.vegh'
```

## Inputs

Inputs can be passed using the `with:` context in your workflow step.

*   `command` (Required): The Vegh command to run. Options: `snap`, `loc`, `send`, `restore`, `audit`, `prompt`, `check`. (Default: `snap`)
*   `path` (Required): The source directory or file path. (Default: `.`)
*   `output` (Optional): Output filename for `snap` or destination folder for `restore`. (Default: `artifact.vegh`)
*   `token` (Optional): Auth token for the `send` command.
*   `url` (Optional): Target URL for the `send` command.
*   `comment` (Optional): Message for the snapshot metadata (`snap`). (Default: `Snapshot via GitHub Actions`)
*   `dry-run` (Optional): Simulate without creating files (`snap`). (Default: `false`)
*   `python-version` (Optional): Python version to use. (Default: `3.12`)
*   `clean` (Optional): Clean mode for `prompt` (remove locks/secrets). (Default: `true`)
*   `sloc` (Optional): Count Source Lines of Code (exclude comments/blanks) for `loc`. (Default: `false`)
*   `extra_args` (Optional): Extra arguments to pass to the underlying `vegh` command. (Default: ``)

## Outputs

*   `loc-report`: The raw LOC report (useful for PR comments).
*   `snapshot-path`: Path to the generated snapshot.
*   `context-path`: Path to the generated AI context XML file (from `prompt` command).

# License

[MIT](LICENSE)
