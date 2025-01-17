# ansible_galaxy_collection

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/artis3n/ansible_galaxy_collection/Testing%20the%20Action)](https://github.com/artis3n/ansible_galaxy_collection/actions)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/artis3n/ansible_galaxy_collection)](https://github.com/artis3n/ansible_galaxy_collection/releases)
![GitHub last commit](https://img.shields.io/github/last-commit/artis3n/ansible_galaxy_collection)
![GitHub](https://img.shields.io/github/license/artis3n/ansible_galaxy_collection)
[![GitHub followers](https://img.shields.io/github/followers/artis3n?style=social)](https://github.com/artis3n/)
[![Twitter Follow](https://img.shields.io/twitter/follow/artis3n?style=social)](https://twitter.com/Artis3n)

Deploy a Collection to Ansible Galaxy.

## Requirements

This action expects to be run from a repository with certain met conditions.

1. The repository contains a valid Ansible Galaxy Collection, meaning it minimally contains a `galaxy.yml` file and a `README.md`.

An example workflow using this action can be found [here](https://github.com/artis3n/ansible-collection-github/blob/master/.github/workflows/deploy.yml) and in [the tests](.github/workflows/main.yml).

## Inputs

### api_key

**Required**: Ansible Galaxy API key.

This should be stored in a Secret on GitHub. See [Creating and Using Secrets Encrypted Variables](https://help.github.com/en/github/automating-your-workflow-with-github-actions/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables).

### collection_dir

**Default**: `./`

The directory in which the Ansible Collection is stored. This defaults to the project root.

Only change this if your Collection is not stored in your project root.

### galaxy_version

Semver-compatible string: `1`, `1.1`, `1.1.1`, `1.1.1-alpha`

Dynamically inject a semver-compatible version into your `galaxy.yml` file.

<!--
This parameter is not compatible with the `galaxy_version_increment` parameter.

### galaxy_version_commit (TBA)

Values: `true`/`false`

The Action will write the `galaxy_version` value into your `galaxy.yml` and commit it.

### galaxy_version_increment (TBA)

Values: `major`/`minor`/`patch`
Default: `patch`

The Action will read the version in galaxy.yml and increment it based on the value provided in this parameter.

This parameter is not compatible with the `galaxy_version` parameter.

-->

### galaxy_config_file (Deprecated)

**Default**: `galaxy.yml`

A collection must have a galaxy.yml file that contains the necessary information to build a collection artifact. Defaults to "galaxy.yml" in the `collection_dir`.

This parameter is deprecated as Ansible Galaxy requires the file to be named `galaxy.yml` and to exist in the root of your Collection. Use `collection_dir` to specify a non-root directory for your Collection.

## Example Usage

Default usage:

```yaml
- name: Build and Deploy Collection
  uses: artis3n/ansible_galaxy_collection@v2
  with:
    api_key: '${{ secrets.GALAXY_API_KEY }}'
```

Pass in `galaxy.yml` version as an input parameter:

```yaml
- name: Get the version name from the tags
  run: echo "RELEASE_VERSION=${GITHUB_REF/refs\/tags\//}" >> $GITHUB_ENV

- name: Injecting a dynamic Collection version
  uses: artis3n/ansible_galaxy_collection@v2
  with:
    api_key: '${{ secrets.GALAXY_API_KEY }}'
    galaxy_version: '${{ env.RELEASE_VERSION }}'
```

If your Collection root is not in your repo root:

```yaml
- name: When the Collection is not in the project root
  uses: artis3n/ansible_galaxy_collection@v2
  with:
    api_key: '${{ secrets.GALAXY_API_KEY }}'
    collection_dir: 'src/my_collection'
```

Not the common way to invoke actions, but save ~3 minutes in your invocation by using the built image instead of building ansible during the image creation. You will need to pin to a specific tag, not a general value like `v2`:

```yaml
- name: Build and Deploy Collection
  uses: docker://ghcr.io/artis3n/ansible_galaxy_collection@v2.4.2
  with:
    api_key: '${{ secrets.GALAXY_API_KEY }}'
```

## Exit Codes

- `0`: `Ok`
- `1`: `DeployFailed`
    - The Action attempted to deploy to Ansible Galaxy, but failed. There will be details of the error from Ansible Galaxy recorded at the end of the action's logs.
- `2`: `ValidationFailed`
    - One or more user-supplied input parameters were invalid. See the error message for validation details.
