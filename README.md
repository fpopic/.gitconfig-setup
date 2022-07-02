# .gitconfig-setup

# How to prepare your git environment

- Generate new [SSH private key(s)](https://docs.gitlab.com/ee/user/ssh.html#generate-an-ssh-key-pair)
  - Generating a private key with a **passphrase** is a requirement on Mac :warning:
    ```shell
    ssh-keygen -t rsa -b 4096 -C "your@github.email" -f ~/.ssh/id_rsa_github
    pbcopy ~/.ssh/id_rsa_github.pub
    open https://github.com/settings/keys
    ```
  - [Make keys "persistent"](https://unix.stackexchange.com/a/560404/171941) automatically loaded after Mac reboot
  - `~/.ssh/config`
    ```
    Host *
        UseKeychain yes
        AddKeysToAgent yes
        IgnoreUnknown UseKeychain

    Host gitlab.com
        identityfile = ~/.ssh/id_rsa_gitlab
        user = git

    Host github.com
        identityfile = ~/.ssh/id_rsa_github
        user = git
    ```
  - For GitHub make sure you authorized your ssh-key with [your organization](https://docs.github.com/en/enterprise-cloud@latest/authentication/authenticating-with-saml-single-sign-on/authorizing-an-ssh-key-for-use-with-saml-single-sign-on)
- Generate new [GPG signing key(s)](https://docs.gitlab.com/ee/user/project/repository/gpg_signed_commits/)
- Note your GPG key ID, It begins after the `/` character in the sec paragraph when executing the command `gpg --list-secret-keys --keyid-format LONG <EMAIL>`
- Set up `~/.gitconfig` so that it automatically picks up your git config based on the folder prefix
  - `~/.gitconfig` (we suggest to follow folder paths: ~/Projects/gitlab/my-organisation1 and ~/Projects/github/my-organisation2 for all cloned repositories)
    ```ini
    [core]
    editor = nano

    [commit]
    gpgsign = true

    [includeIf "gitdir:~/Projects/gitlab/my-organisation1/"]
    path = ~/.gitconfig-gitlab-my-organisation1

    [includeIf "gitdir:~/Projects/github/my-organisation2/"]
    path = ~/.gitconfig-github-my-organisation2
    ```

  - `~/.gitconfig-gitlab-my-organisation1`
    ```ini
    [user]
    email = first.last@organisation1.com
    name = your-gitlab-username
    signingkey = your GPG key ID
    ```

  - `~/.gitconfig-github-my-organisation2`
    ```ini
    [user]
    email = your@github.email
    name = your-github-username
    signingkey = your GPG key ID
    ```

- Make sure everything works by cloning some repository using git ssh protocol

## Troubleshooting
- If you get the error error: `gpg failed to sign the data` try running `export GPG_TTY=$(tty)` before commiting and add it to your `~/.zshrc` config if it helped

```shell
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
source ~/.zshrc
```
