# GitHub Actions Matrix

## Unidimensional matrix

You can use GitHub Actions' matrix to run a job multiple times with different parameters.

I'm only listening for push events for the demo, but it works with every events.

```
name: CI
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [11, 12, 13, 14]
    steps:
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node }}
      - name: Node version
        run: node -v
```
CI: https://github.com/Crow-EH/matrix-sandbox/actions/runs/86621332

## Multidimensional matrix

If you put multiple parameters in the matrix, a job will be run for every possible combination.

```
name: CI
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [11, 12, 13, 14]
        chalk: [3, 4]
    steps:
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node }}
      - name: Node version
        run: node -v
      - name: Install chalk
        run: npm install -g chalk@${{ matrix.chalk }}
      - name: Chalk version
        run: npm list -g chalk
```
CI: https://github.com/Crow-EH/matrix-sandbox/actions/runs/86630358

## Runs-on parameter

You can of course configure the OS to run your jobs on.

```
name: CI
on: push
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [macos-latest, windows-latest, ubuntu-latest]
        node: [11, 12, 13, 14]
        chalk: [3, 4]
    steps:
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node }}
      - name: Node version
        run: node -v
      - name: Install chalk
        run: npm install -g chalk@${{ matrix.chalk }}
      - name: Chalk version
        run: npm list -g chalk
```
CI: https://github.com/Crow-EH/matrix-sandbox/actions/runs/86632999

## Inclusion and exclusion

With `include`, you can add new parameters in some existing combinations, or create a new specific combination.

**Caution**: For an unknown reason, if you have a parameter for `runs-on`, it must be included in all your `include` items (if not, GitHub Actions will throw an error).

With `exclude`, you can remove some combinations.

**Caution**: `include` is interpreted after `exclude`

```
name: CI
on: push
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [macos-latest, windows-latest, ubuntu-latest]
        node: [11, 12, 13, 14]
        chalk: [3, 4]
        include:
          - os: ubuntu-latest
            node: 11
            chalk: 2
        exclude:
          - os: windows-latest
            node: 12
          - os: windows-latest
            node: 13
          - os: windows-latest
            node: 14
          - os: windows-latest
            chalk: 3
    steps:
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node }}
      - name: Node version
        run: node -v
      - name: Install chalk
        run: npm install -g chalk@${{ matrix.chalk }}
      - name: Chalk version
        run: npm list -g chalk
```
CI: https://github.com/Crow-EH/matrix-sandbox/actions/runs/86640668

## The undocumented way: Objects array

Albeit undocumented, instead of using the combination mechanism, you can define an array of objects.

You can of course use both styles at the same time.

```
name: CI
on: push
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        config:
          - { node: 11, chalk: 2 }
          - { node: 12, chalk: 3 }
          - { node: 13, chalk: 3 }
          - { node: 14, chalk: 4 }
        os: [macos-latest, ubuntu-latest]
        include:
          - config: { node: 11, chalk: 2 }
            os: windows-latest
    steps:
      - name: Setup node
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.config.node }}
      - name: Node version
        run: node -v
      - name: Install chalk
        run: npm install -g chalk@${{ matrix.config.chalk }}
      - name: Chalk version
        run: npm list -g chalk
```
CI: https://github.com/Crow-EH/matrix-sandbox/actions/runs/86678840

## Check requirement and its behavior with matrix jobs

If needed, you can require the jobs run by GitHub Actions to be successful before merging any Pull Request.

However, there's a specific behavior / limitation when dealing with matrix jobs: There is no way to require the whole matrix in one go, so you must require every existing combinations separately.

While this could be alright for simple / rarely evolving matrices, when your matrix changes often it will rapidly become a huge problem: Every time anyone changes, remove or add a matrix job, a repository owner will have to update the checks requirements.

If you want to automatically check for every possible matrix jobs, you'll have to create a custom workflow whose only job would be to check the other jobs.

Luckily, GitHub Actions has an API, and a lot of actions already do what we want.

On [ekino/docker-buildbox](https://github.com/ekino/docker-buildbox), we created a new workflow using [WyriHaximus/github-action-wait-for-status](https://github.com/WyriHaximus/github-action-wait-for-status) to check for all the other jobs' statuses, and required only this new job.

Once again, I'm listening to push events for the demo, but you should use this workflow for pull requests only.

```
name: Build Checker
on: push
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - name: Wait for commit statuses
        id: status
        uses: WyriHaximus/github-action-wait-for-status@d2ddfe5
        with:
          ignoreActions: check
          checkInterval: 60
        env:
          GITHUB_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
      - name: Succeed
        if: steps.status.outputs.status == 'success'
        run: exit 0
      - name: Fail
        if: steps.status.outputs.status == 'failure'
        run: exit 1
```
CI: https://github.com/Crow-EH/matrix-sandbox/actions/runs/86690947

Build Checker: https://github.com/Crow-EH/matrix-sandbox/actions/runs/86690945
