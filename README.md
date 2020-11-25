# Remove old package versions from GitHub Packages

This action will allow you to remove older versions of all GitHub Packages in a private repository.

## Usage

To use the action, simply refer to it in your workflow:

```yaml
name: Remove package versions
on: push
jobs:
  remove-package-versions:
    runs-on: ubuntu-latest
    steps:
    - name: Remove package versions
      id: remove-package-versions
      uses: navikt/remove-package-versions@v0
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The above example will trigger the action whenever the `push` event occurs. All events that can be used are found in the [GitHub documentation](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows).

If you want to run the action periodically instead, use a scheduled workflow:

```yaml
name: Remove package versions
on:
  schedule:
    - cron:  '0 8 * * *'
jobs:
  remove-package-versions:
    runs-on: ubuntu-latest
    steps:
      - name: Remove package versions
        id: remove-package-versions
        uses: navikt/remove-package-versions@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Note:** The underlying HTTP client in this action does not currently do any pagination which means it has a hard limit on 100 packages, and 100 versions per package when fetching the packages / versions to determine what to remove.

## Environment variables

The action uses the following environment variables:

### `GITHUB_TOKEN`

A valid token that is used to interact with GitHubs API. This token must have the following scopes:

- `repo`
- `read:packages`
- `delete:packages`

As seen from the above example the autogenerated installation token within a workflow run can be used, so there is no need to generate a personal access token for this purpose.

### `GITHUB_REPOSITORY`

This environment variable is automatically available inside a workflow run, and the format is `owner:repo`, for instance `navikt/remove-package-versions`.

## Parameters

The action supports the following parameters:

### `keep_versions`

Number of versions to keep per package. Defaults to `5`.

### `keep_latest`

Wheter or not to keep the `latest` version. Defaults to `true`.

### `remove_semver`

Whether or not to remove [semantic versions](https://semver.org/). Defaults to `false`.

Keep in mind that if you use date-based versions with `.` as a separator, for instance `yyyy.mm.dd-<sha>`, some dates are valid semantic versions, while others aren't. The ones who aren't valid semantic versions are the ones who have the month or day parts of the date starting with `0`, for instance `2019.12.01`. When using date-based versions it's advisable to use `-` as a separator instead of `.`.

## Output

The action outputs a JSON-encoded list of removed package versions prefixed with the owner and the repository name. The name of the output is `removed_package_versions`. Using the `id` from the workflow example above, you can refer to the output using `${{ steps.remove-package-versions.outputs.removed_package_versions }}`.

## Releasing a new version

Use the `release.sh` script to easily tag the `main` branch:

```
./release.sh v1.2.3
```

will create three tags, all pointing to `HEAD`:

- `v1`
- `v1.2`
- `v1.2.3`

The script will also ask if you want to push the tags. Existing major and/or minor tags will be replaced.

## License

MIT, see [LICENSE](LICENSE).
