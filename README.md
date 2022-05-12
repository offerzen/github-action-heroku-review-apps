# GitHub Action - Manage Heroku Review Apps

This GitHub action manages Heroku review apps from Github.

### Features

- Create a review app by adding a label called `create-review-app`.
- Update a review app by adding a label called `update-review-app`.
- Delete a review app by adding a label called `delete-review-app`.

## How To Use

Here's an example workflow that uses this action. This example workflow runs when you add labels to an open Pull Request.

```yaml
name: Review App

on:
  pull_request:
    types: [labeled]

jobs:
  review-app:
    runs-on: ubuntu-latest
    name: ${{ github.event.label.name }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Review App
        uses: offerzen/github-action-heroku-review-apps@v1
        id: deploy
        with:
          api-key: ${{ secrets.HEROKU_API_KEY }}
          pipeline-id: <pipeline ID>
          base-name: <base name>
```

The action expects a few input parameters which are defined below.

- **api-key:** _(required)_ Your Heroku API key.
- **pipeline-id:** _(required)_ The id of the pipeline for the review apps.
- **base-name:** _(required)_ The prefix used to generate review app name. This should be the same as what you specified for the review app URL pattern in Heroku Dashboard.

Optional, but advised, is to:
1. Automatically remove labels.
1. Post a comment on the PR with the link to the review app.
1. Notify in a slack channel if the workflow fails.

The full workflow then becomes:
```yaml
name: Review App

on:
  pull_request:
    types: [labeled]

jobs:
  review-app:
    runs-on: ubuntu-latest
    name: ${{ github.event.label.name }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Review App
        uses: offerzen/github-action-heroku-review-apps@v1
        id: deploy
        with:
          api-key: <Heroku API key>
          pipeline-id: <Pipeline ID>
          base-name: <Prefix for review app name>

      - name: Remove label
        uses: fastruby/pr-unlabeler@v2
        with:
          label-to-remove: ${{ github.event.label.name }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Get the review app url
        run: echo "The URL is ${{ steps.deploy.outputs.url }}"

      - name: Comment on commit
        uses: phulsechinmay/rewritable-pr-comment@v0.2.1
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          COMMENT_IDENTIFIER: "Deploy"
          message: |
            Review App: ${{ steps.deploy.outputs.url }}

      - name: Notify
        if: ${{ failure() }}
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
        env:
          SLACK_WEBHOOK_URL: <Slack webhook URL>
```

## Versions

### Version 1
Uses Pull Request actions to manage the review app:
- On PR open or reopen: creates the review app
- On PR synchronize: updates the review app
- On PR close: deletes the review app

Requires the following workflow triggers:
```yaml
on:
  pull_request:
    types: [opened, reopened, closed, synchronize]
```

Installed using:
```yaml
uses: offerzen/github-action-heroku-review-apps@v1
```

### Version 2
Uses labels to manage the review app:
- When adding a label called `create-review-app`: creates the review app
- When adding a label called `update-review-app`: updates the review app
- When adding a label called `delete-review-app`: deletes the review app

Requires the following workflow trigger:
```yaml
on:
  pull_request:
    types: [labeled]
```

Installed using:
```yaml
uses: offerzen/github-action-heroku-review-apps@v2
```
