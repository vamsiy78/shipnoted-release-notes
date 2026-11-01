# Shipnoted release notes (GitHub Action)

When you push a tag, this writes your release notes from the pull requests merged since the previous tag, and files them in Shipnoted as a draft (or publishes them).
Merged pull requests that mention a request (`SN-42`) close the loop: the request moves to **Shipped** and the people who voted for it get an email.

```yaml
# .github/workflows/release-notes.yml
name: Release notes
on:
  push:
    tags: ["v*"]
jobs:
  notes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: vamsiy78/shipnoted-release-notes@v1
        with:
          api-key: ${{ secrets.SHIPNOTED_API_KEY }}
          project: your-slug
          # publish: "true"     # skip the review step
          # email: "true"       # and email your subscribers
```

Create the key under **Install & connect → API keys** with the **CI** preset (read and write releases). It needs nothing else.

| Input | Default | |
| --- | --- | --- |
| `api-key` | | Required. A key with the `releases:write` scope. |
| `project` | | Required. Your project's slug. |
| `tag` | the tag that triggered the run | Becomes the release's version. |
| `previous-tag` | the tag before `tag` | Where the notes start. |
| `publish` | `false` | `true` publishes at once. Otherwise you get a draft to review. |
| `email` | `false` | With `publish`, also email confirmed subscribers. |
| `title` | written from the changes | |
| `url` | `https://shipnoted.com` | Only for a custom domain. |

Outputs: `release-id`, `url` (when published) and `source` (`claude` or `builtin`, whoever wrote the notes).

No changes since the last tag is not an error: the step ends quietly and creates nothing.

The action is a thin wrapper around `npx shipnoted release draft`, so you can run the same thing anywhere: `SHIPNOTED_API_KEY=… SHIPNOTED_PROJECT=… npx shipnoted release draft --tag v1.2.0 --publish`.
