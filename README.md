# LABORATORY-PIPELINE

```bash
$ git update-index --chmod=+x ./.github/scripts/hello.sh
```

```bash
curl -X POST -H "Accept: application/vnd.github+json" -H "Authorization: Bearer <BEARER TOKEN>" "https://api.github.com/repos/justalk/LABORATORY-PIPELINE/actions/workflows/dispatch.yml/dispatches" -d '{"ref":"feature/check-github-pipeline","inputs":{"testVariable":"KEVIN", "environment":"stagging"}}'
```
