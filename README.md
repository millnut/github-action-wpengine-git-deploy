# GitHub Action for WP Engine Git Deployments

An action to deploy your repository to a **[WP Engine](https://wpengine.com)** site via git. [Read more](https://wpengine.com/git/) about WP Engine's git deployment support.

**Don't** forget to set `fetch-depth` to 0 as in the file example below because WPE rejects shallow stuff and won't take your push. I spent hours to figure that out and tell you one sentence.

## Example GitHub Action workflow

```
name: CI

on:
  push:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repo
        uses: actions/checkout@v1
        with:
         fetch-depth: 0 # WPE rejects shallow pushes so we need to tell checkout to get all history
        
      - name: Deploy to WP-Engine
        uses: curtismchale/github-action-wpengine-git-deploy@8 # Note, the `@8` is the release name use @master if you want the master branch of this action
        env:
          WPENGINE_ENVIRONMENT_NAME: hotsauce # actually see the description below because you need to change this to your environment name.
          WPENGINE_SSH_KEY_PRIVATE: ${{ secrets.WPENGINE_SSH_KEY_PRIVATE }} # Configured in Repo/Settings/Secrets
          WPENGINE_SSH_KEY_PUBLIC:  ${{ secrets.WPENGINE_SSH_KEY_PUBLIC }} # Configured in Repo/Settings/Secrets
```

## Environment Variables & Secrets

### Required

| Name | Type | Usage |
|-|-|-|
| `WPENGINE_ENVIRONMENT_NAME` | Environment Variable | The name of the WP Engine environment you want to deploy to. You can find that under the Git Push settings in your WPE dashboard. It's whatever comes just before .git under the production repository information. So if your repository is `git@git.wpengine.com:production/hotsauce.git` then `hotsauce` goes here.  |
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
