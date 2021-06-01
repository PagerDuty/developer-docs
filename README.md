# Work in progress, these aren't actually deployed. Ignore the following docs and go direct to github.com/pagerduty/developer-site to update the docs.

# Developer Docs

This repo contains the docs that are hosted at: [developer.pagerduty.com/docs](https://developer.pagerduty.com/docs)

## Contributing

**Everyone** at PagerDuty encouraged to write and modify Developer Docs. Simply edit the files and open a pull request.

### Ownership
These docs are owned by the #dev-infra team and they will review and deploy any changes made- simply ask them via Slack when your open a pull request.

## Markdown 
The docs are written in simple markdown format with some flavour on top that [Stoplight](stoplight.io) provides to us. 

  - [Markdown Formatting](https://www.markdownguide.org/basic-syntax/)
  - [Stoplight Markdown]( https://meta.stoplight.io/docs/studio/docs/Documentation/03a-stoplight-flavored-markdown.md)

## Editing the "Sidebar"/Table of Contents
Documents are hidden by default, to provide a link to a document via the Table of Contents you must edit the Table of Contents file: `toc.json`

Documentation on how `toc.json` works can be [found here.](https://meta.stoplight.io/docs/platform/4.-documentation/d.table-of-contents.md)

## File Naming
There isn't a strict file naming convention. *However*, files have broadly been organized into 'topic' folder, and file names begin with the order they appear in the Table of Contents.

## Deployment
Simply merging to the `main` branch will deploy docs to production.

**There is no staging at the moment**