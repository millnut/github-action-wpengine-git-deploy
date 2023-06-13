# GitHub Action for WP Engine Git Deployments

This GitHub Action automates the deployment of your WordPress website to WP Engine using Git. It handles setting up the SSH keys, configuring the Git remote, and pushing your changes to the WP Engine environment. [Read more](https://wpengine.com/git/)

This image is an update fork of https://github.com/jovrtn/github-action-wpengine-git-deploy.

## Please note
Don't forget to set `fetch-depth` to 0 as in the file example below because WP Engine rejects shallow stuff and won't take your push. I spent hours to figure that out and tell you one sentence.

## Usage
To use this GitHub Action in your workflow, add the following steps to your `.github/workflows/main.yml`  :

```yaml
name: CI

on:
  push:
    branches: main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          fetch-depth: 0 # WP Engine rejects shallow pushes so we need to tell checkout to get all history

      - name: Deploy to WP Engine
        uses: millnut/github-action-wpengine-git-deploy@main
        env:
          WPENGINE_ENVIRONMENT_NAME: ${{ secrets.WPENGINE_ENVIRONMENT_NAME }}
          WPENGINE_SSH_KEY_PRIVATE: ${{ secrets.WPENGINE_SSH_KEY_PRIVATE }}
          WPENGINE_SSH_KEY_PUBLIC: ${{ secrets.WPENGINE_SSH_KEY_PUBLIC }}
          LOCAL_BRANCH: main
```

## Environment Variables & Secrets

### Required

| Name | Type | Usage |
|-|-|-|
| `WPENGINE_ENVIRONMENT_NAME` | Environment Variable | The name of the WP Engine environment you want to deploy to. |
| `WPENGINE_SSH_KEY_PRIVATE` | Secret | Private SSH key of your WP Engine git deploy user. See below for SSH key usage. |
|  `WPENGINE_SSH_KEY_PUBLIC` | Secret | Public SSH key of your WP Engine git deploy user. See below for SSH key usage. |

### Optional

| Name | Type  | Usage |
|-|-|-|
| `WPENGINE_ENVIRONMENT` | Environment Variable  | Defaults to `production`. You shouldn't need to change this, but if you're using WP Engine's legacy staging, you can override the default and set to `staging` if needed. |
| `LOCAL_BRANCH` | Environment Variable  | Set which branch in your repository you'd like to push to WP Engine. Defaults to `master`. |

### Further reading

* [Defining environment variables in GitHub Actions](https://developer.github.com/actions/creating-github-actions/accessing-the-runtime-environment/#environment-variables)
* [Storing secrets in GitHub repositories](https://developer.github.com/actions/managing-workflows/storing-secrets/)

## Setting up your SSH keys

1. [Generate a new SSH key pair](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) as a special deploy key. The simplest method is to generate a key pair with a blank passphrase, which creates an unencrypted private key.
2. Store your public and private keys in your GitHub repository as new 'Secrets' (under your repository settings), using the names `WPENGINE_SSH_KEY_PRIVATE` and `WPENGINE_SSH_KEY_PUBLIC` respectively. In theory, this replaces the need for encryption on the key itself, since GitHub repository secrets are encrypted by default.
3. Add the public key to your target WP Engine environment.
4. Per the [WP Engine documentation](https://wpengine.com/git/), it takes about 30-45 minutes for the new SSH key to become active.
