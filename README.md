# GitHub Actions Matrix

## Unidimensional matrix

You can use GitHub Actions' matrix to run a job multiple times with different parameters.

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
https://github.com/Crow-EH/matrix-sandbox/actions/runs/86621332

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
https://github.com/Crow-EH/matrix-sandbox/actions/runs/86630358

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
https://github.com/Crow-EH/matrix-sandbox/actions/runs/86632999

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
https://github.com/Crow-EH/matrix-sandbox/actions/runs/86632999

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
https://github.com/Crow-EH/matrix-sandbox/actions/runs/86632999
