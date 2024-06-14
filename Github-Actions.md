# Github Actions

## Action

### Using LFS objects

```yaml
runs:
  steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        lfs: true
    - name: Checkout LFS objects
      run: git lfs checkout
```

### `shell` property required in `runs.steps.*` if `run` property used

```yaml
runs:
  using: "composite"
  steps:
    - id: do-something
      shell: bash
      run: |
        echo "do-something"
```

### How to define outputs

```yaml
outputs:
  foo:
    value: ${{ steps.do-something.outputs.foo }}runs:

runs:
  using: "composite"
  steps:
    - id: do-something
      shell: bash
      run: |
        $foo="some-inline-value"
        echo "foo=$foo" >> "$GITHUB_OUTPUT"
```

### How to write logs or create file annotation

* https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-a-notice-message
* https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-a-warning-message
* https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-error-message


```yaml
runs:
  using: "composite"
  steps:
    - name: Raise an error
      if: ${{ steps.get_changelog.outputs.changes == '' }}
      shell: bash
      run: |
        echo "::{notice|warning|error} file={filename},line={line},col={col},endColumn={endColumn},endLine={endLine},title={title}::{message}"
        exit 1
```

Default values
```
::{notice|warning|error} file=.github,line=1,col={col},endColumn={endColumn},endLine=1,title={title}::{message}
```

#### Usage

```
::{notice|warning|error} file={filename},line={line},col={col},endColumn={endColumn},endLine={endLine},title={title}::{message}
::{notice|warning|error} file={filename}::{message}
::{notice|warning|error} title={title}::{message}
```

#### Output

##### Summary
```
{title}: {filename}#{endLine or line}
{message}
```

##### Logs
```
{notice|warning|error} {message}
```

#### File Annotation
```
Check {notice|warning|failure} on line {endLine or line} in {filename}
GitHub Actions / {job.name}
{title or filename#L(endLine or line)}
{message}
```

### How to throw error

* https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-error-message

Using `::error` command and `exit 1`

```yaml
runs:
  using: "composite"
  steps:
    - name: Raise an error
      if: ${{ steps.get_changelog.outputs.changes == '' }}
      shell: bash
      run: |
        echo "::error file={filename},line={line},col={col},endColumn={endColumn},endLine={endLine},title={title}::{message}"
        exit 1
```

### How to catch an error

* https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context

Using `continue-on-error` and steps context `outcome`.
When a `continue-on-error` step fails, the `outcome` is `failure`, but the final `conclusion` is `success`.

```yaml
runs:
  using: "composite"
  steps:
    - name: Raise an error
      id: try-action
      continue-on-error: true
      shell: bash
      run: |
        exit 1

    - name: Catch the error
      if: ${{ steps.try-action.outcome == 'failure' }}
      shell: bash
      run: |
        echo "::warning some useful description"
        exit 1
```
