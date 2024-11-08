# Review Pull Request with Gemini API

<p>
<a href="https://bunhere.com"><img src="./images/reviewpr.png" width="24" alt="WTM"></a>
<a href="#"><img src="https://img.shields.io/badge/Review PR-v1.0.0-blue" alt="Latest Stable Version"></a>
<a href="#"><img src="https://img.shields.io/badge/license-MIT-yellow" alt="License"></a>
</p>

## Quick Start

### 1. Create `package.json`

```json
{
  "name": "Project Name",
  "main": "index.html",
  "devDependencies": {
    "@types/node": "^22.9.0"
  },
  "dependencies": {
    "@actions/core": "^1.11.1",
    "@actions/github": "^6.0.0",
    "@google/generative-ai": "^0.21.0",
    "axios": "^1.7.7",
    "dotenv": "^16.4.5",
    "loglevel": "^1.9.2",
    "tslog": "^4.9.3"
  }
}
```

### 2. Create `GEMINI_API_KEY` and `GIT_TOKEN_KEY`

After having the 2 above keys, at the github repository, go to **Settings** > **Secrets and Variales** > **Actions** > ***Add 2 new secret keys***.

### 3. Create `.github/workflows/[gemini-review-code].yml`

```yml
name: "Review the code with Gemini"

on:
  pull_request:

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v3

      - name: Install dependencies
        run: npm install

      - name: "Get diff of the pull request"
        id: get_diff
        shell: bash
        env:
          PULL_REQUEST_HEAD_REF: "${{ github.event.pull_request.head.ref }}"
          PULL_REQUEST_BASE_REF: "${{ github.event.pull_request.base.ref }}"
        run: |-
          git fetch origin "${{ env.PULL_REQUEST_HEAD_REF }}"
          git fetch origin "${{ env.PULL_REQUEST_BASE_REF }}"
          git checkout "${{ env.PULL_REQUEST_HEAD_REF }}"
          git diff "origin/${{ env.PULL_REQUEST_BASE_REF }}" > "diff.txt"
          {
            echo "pull_request_diff<<EOF";
            cat "diff.txt";
            echo 'EOF';
          } >> $GITHUB_OUTPUT

      - name: "Review the code with Gemini"
        uses: ./
        env: # Set environment variables here
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GIT_TOKEN_KEY }}
          GITHUB_REPOSITORY: ${{ github.repository }}
          GITHUB_PULL_REQUEST_NUMBER: ${{ github.event.pull_request.number }}
          GIT_COMMIT_HASH: ${{ github.event.pull_request.head.sha }}
        with:
          model: "gemini-1.5-pro-latest"
          pull_request_diff: |-
            ${{ steps.get_diff.outputs.pull_request_diff }}
          pull_request_chunk_size: "3500"
          extra_prompt: ""
          log_level: "DEBUG"
```