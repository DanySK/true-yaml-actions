# True YAML Actions

I gave up hope of seeing [actions/runner#1182](https://github.com/actions/runner/issues/1182) fixed within my lifespan¹.
I thus decided to write a very simple composite GitHub action that adds support for YAML anchors and merge keys in GitHub Actions.

## How to use it

Write your workflows into `.github/real-workflows`,
freely using merge keys (`<<:`) and anchors (`*`),
but writing all your anchors definitions in a header document,
then write the real workflow separated by `---`, like this:

```yaml
anchor: &anchor
  name: Test running on ${{github.repository}}
  run: echo Commit ${{github.sha}} seems to be working

---

name: Test this action

on:
  push:
  pull_request:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Test running on ${{github.repository}}
        run: echo Commit ${{github.sha}} seems to be working
      - <<: *anchor
        run: echo Test merge keys
      - *anchor
```

Now, create a new token with `workflows:write` permissions and register it as a repo (or organization) secret.
In the remainder, we'll call this token's secret `WORKFLOW_TOKEN`.

Now, create an additional workflow in `.github/workflows/autogen.yaml`
(of course pick the name you like the most, the important thing is the workflow location)
that looks like this:

```yaml
name: Generate workflows

on:
  push:
    branches:
      - master

jobs:
  generate-workflows:
    runs-on: ubuntu-latest
    steps:
      - name: Run locally
        uses: 'DanySK/true-yaml-actions@master'
        with:
          token: ${{ secrets.WORKFLOW_TOKEN }}

```

Replace `WORKFLOW_TOKEN` with the name of the secret you created before.

That's it.

![image](https://github.com/DanySK/true-yaml-actions/assets/1991673/ed6631c0-22a7-41bf-bff3-603aa7dfc434)


1. I also honestly believe that GitHub Actions was built bottom-up, has great design flaws, and deserves a v2, but still, it's the best thing we have, so I can't help but love it ❤️
