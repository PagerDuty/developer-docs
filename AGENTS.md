# AGENTS.md

This repo holds the PagerDuty developer docs. The docs are published at [developer.pagerduty.com/docs](https://developer.pagerduty.com/docs) through [Stoplight](https://stoplight.io). The repo has no application code, no build step and no test suite.

## Layout

- `docs/` holds the articles as Stoplight-flavored Markdown. Each topic has its own folder, for example `docs/webhooks/` or `docs/REST-API/`. File names start with a number that gives the order in the sidebar.
- `assets/images/` holds the images that articles reference.
- `toc.json` defines the sidebar navigation. Each `item` has a `title`, a `uri` that points to a file in `docs/`, and a `slug`. A `group` holds a list of items.
- `.stoplight.json` lists the files that the Stoplight push must not publish.
- `.buildkite/stg-pipeline.yml` pushes a pull request to the staging site.

Stoplight hides a document from the sidebar until `toc.json` links to it. When you add an article, add an entry to `toc.json`. When you rename or move an article, update its `uri` in `toc.json`. Keep the `slug` unchanged when you can, because the slug is the public URL.

## How to write a change

1. Edit or add a Markdown file in `docs/`. Use [Stoplight Markdown](https://meta.stoplight.io/docs/studio/docs/Documentation/03a-stoplight-flavored-markdown.md) for callouts, tabs and code groups.
2. Put new images in `assets/images/` and reference them with a relative path.
3. Update `toc.json` if the sidebar must change. Make sure that `toc.json` is still valid JSON: `python3 -m json.tool toc.json > /dev/null`.
4. Check that each `uri` in `toc.json` points to a file that exists.

## How to preview a change

There is no local preview. Open a pull request. In the Buildkite build for the pull request, unblock the "Deploy to Staging" step. The pipeline runs `npx @stoplight/cli@5 push` and the change shows on the [staging site](https://developer-v2.pd-staging.com/docs).

## How a change reaches production

Merge the pull request to `main`. A merge to `main` deploys the docs to production. The Dev Ecosystem and Public APIs team (#dev-ecosystem-and-public-apis) owns the repo and reviews every change. Changes also need a review from Community, as the pull request template says.

## Agent and repo files

Do not let repo tooling files publish as articles. When you add a file that is not an article (for example agent config or CI config), add it to the `exclude` list in `.stoplight.json`.

## More docs

See [docs/README.md](docs/README.md) for the index of repo docs.
