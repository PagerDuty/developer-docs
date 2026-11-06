# Developer Docs (retired)

> **This repo is retired. Do not open pull requests here.**
> The content of this repo now lives in the [pagerduty-docs](https://github.com/PagerDuty/pagerduty-docs) repo and is published at https://docs.pagerduty.com.

## What changed

- https://developer.pagerduty.com/docs is no longer served from this repo.
- The pagerduty-docs redirector serves developer.pagerduty.com. It sends legacy `/docs/...` paths (301) to their new pages on docs.pagerduty.com.
- Merges to `main` do not deploy anything.
- The files in `docs/`, `assets/` and `toc.json` stay here only as history. They are not maintained.

## How to change the docs

1. Find the page on https://docs.pagerduty.com.
2. Edit its source in [pagerduty-docs](https://github.com/PagerDuty/pagerduty-docs) and open a pull request there.
3. For questions, ask the Dev Ecosystem and Public APIs team in [#ask-dev-ecosystem-and-public-apis](https://pagerduty.enterprise.slack.com/archives/C08E4D3RBJQ).

## Related

- [developer-site](https://github.com/PagerDuty/developer-site) is retired too.
- [captains-log](https://github.com/PagerDuty/captains-log) and [api-schema](https://github.com/PagerDuty/api-schema) stay active in their own repos.
