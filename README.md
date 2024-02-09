
# How to use [hype](https://github.com/gopherguides/hype) as an action to auto generate your repo's README.

This repo shows how to use a github action and hype to auto generate your README.md.

## Requirements

For this action to work, you need to either configure your repo with specific permissions, or use a `personal access token`.

### Repo Permissions

You need to give permission to your GitHub Actions to create a pull request in your GitHub repo settings _(Settings -> Actions -> General)_.

Under `Workflow Permissions`


* Check `Allow GitHub Actions to create and approve pull requests`.
* Check `Read and write permissions`


### Personal Access Token

Alternately, you can use tokens to give permission to your action.

It is recommend to use a GitHub [Personnal Acces Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-fine-grained-personal-access-token) like: `${{secrets.PAT}}` instead of using `${{secrets.GITHUB_TOKEN}}` in GitHub Actions.

## The Action

The current action is set to only generate the readme on a pull request and commit it back to that same pull request.  You can modify this to your own needs.

```yml
name: Generate README with Hype
on: [pull_request]

jobs:

  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        repository: ${{ github.event.pull_request.head.repo.full_name }}
        ref: ${{ github.event.pull_request.head.ref }}
    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: "1.22.x"
        cache-dependency-path: subdir/go.sum

    - name: Install hype
      run: go install github.com/gopherguides/hype/cmd/hype@latest

    - name: Run hype
      run: pushd .hype;hype export -format=markdown -f module.md > ../README.md;popd

    - name: Commit README back to the repo
      run: |-
        git rev-parse --abbrev-ref HEAD
        git config user.name 'GitHub Actions'
        git config user.email 'actions@github.com'
        git diff --quiet || (git add README.md && git commit -am "Updated README")
        git push origin ${{github.event.pull_request.head.ref}}

```

