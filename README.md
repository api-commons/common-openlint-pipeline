# API Commons OpenLint Pipeline

**Governance as code.** Drop your [OpenLint](https://openlint.org) ruleset into
CI and every change to your API descriptions is linted against it — as a gate. This
repo is a set of **thin, reusable CI integrations** that all run the
[OpenLint CLI](https://www.npmjs.com/package/openlint) (`npx openlint`) the same way, with the
same inputs (files, ruleset, severity threshold) across platforms.

The unit you "deploy" is **your ruleset + a fail-severity** — see
[`.openlint.example.yaml`](./.openlint.example.yaml). Built-in rulesets are
`spectral:oas`, `spectral:asyncapi` and `spectral:arazzo`; the CLI reads both
`.openlint.yaml` and `.spectral.yaml`.

## GitHub Actions

Use the action directly:

```yaml
# .github/workflows/api-governance.yml
on: [pull_request]
jobs:
  openlint:
    runs-on: ubuntu-latest
    permissions: { contents: read, security-events: write }
    steps:
      - uses: actions/checkout@v4
      - uses: api-commons/common-openlint-pipeline@v1
        with:
          files: 'apis/**/openapi.yaml'
          ruleset: '.openlint.yaml'
          fail-severity: 'error'
          sarif: 'openlint.sarif'           # optional → code scanning
      - if: always()
        uses: github/codeql-action/upload-sarif@v3
        with: { sarif_file: openlint.sarif }
```

| Input | Default | Description |
| --- | --- | --- |
| `files` | `**/{openapi,swagger,asyncapi,arazzo}.{yaml,yml,json}` | Glob(s) to lint |
| `ruleset` | `.openlint.yaml` | Your ruleset (a default extending the built-ins is used if absent) |
| `fail-severity` | `error` | Fail on this severity or worse (`error`/`warn`/`info`/`hint`) |
| `sarif` | _(off)_ | Also write a SARIF report for GitHub code scanning |
| `version` | `latest` | OpenLint CLI (`openlint` on npm) version |

It emits inline **PR annotations** (`--format github-actions`) and a readable summary,
and fails the job per `fail-severity`.

## Other platforms

Copy the matching template from [`templates/`](./templates) — all run the same
`openlint lint … --ruleset … --fail-severity …`, publishing findings natively:

- **GitLab CI** — [`templates/gitlab-ci.yml`](./templates/gitlab-ci.yml) (JUnit report)
- **Bitbucket Pipelines** — [`templates/bitbucket-pipelines.yml`](./templates/bitbucket-pipelines.yml)
- **Azure DevOps** — [`templates/azure-pipelines.yml`](./templates/azure-pipelines.yml) (JUnit)
- **AWS CodeBuild** — [`templates/aws-buildspec.yml`](./templates/aws-buildspec.yml) (JUnit report)

## Agent skills (SKILL.md)

Earlier versions of this pipeline shipped a skill-governance template that linted
`SKILL.md` files. That depended on a markdown/frontmatter parser and a built-in skill
ruleset that the OpenLint CLI does not have (it parses only YAML/JSON documents), so the
template has been removed. API descriptions — OpenAPI, AsyncAPI, Arazzo — are fully governed.

## Part of API Commons

One engine (the OpenLint CLI), one vocabulary (the [spec](https://spec.openlint.org) +
its [rules](https://apicommons.org/rules/)). Author rules in the
[validator](https://validator.apicommons.org), enforce them here. Related:
[common-openlint-validator](https://github.com/api-commons/common-openlint-validator) ·
[common-openlint-api](https://github.com/api-commons/common-openlint-api) ·
[common-openlint-mcp](https://github.com/api-commons/common-openlint-mcp).

---

A project of [API Evangelist](https://apievangelist.com), maintained openly under [API Commons](https://apicommons.org). Apache-2.0.
