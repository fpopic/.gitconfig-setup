# .gitconfig-setup

How to prepare your git environment to work:
- with different auth `ssh` and signing `gpg` keys
- with different git hosts (e.g. GitHub, GitLab) and organization (private account, company account) 
- with automatic keys selection based on the folder path

Steps:
1. Generate new [SSH private key(s)](https://docs.gitlab.com/ee/user/ssh.html#generate-an-ssh-key-pair)
    - Generating a private key with a **passphrase** is a requirement on Mac :warning:
      ```shell
      $ ssh-keygen -t rsa -b 4096 -C "your@gitlab.email" -f ~/.ssh/id_rsa_gitlab
      $ pbcopy ~/.ssh/id_rsa_gitlab.pub
      $ open https://gitlab.com/-/profile/keys

      $ ssh-keygen -t rsa -b 4096 -C "your@github.email" -f ~/.ssh/id_rsa_github
      $ pbcopy ~/.ssh/id_rsa_github.pub
      $ open https://github.com/settings/keys
      ```
    - [Make keys "persistent"](https://unix.stackexchange.com/a/560404/171941) automatically loaded after Mac reboot
    - Update `~/.ssh/config` file:
      ```config
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
    - For GitHub make sure you authorized your ssh key with [your organization via SSO](https://docs.github.com/en/enterprise-cloud@latest/authentication/authenticating-with-saml-single-sign-on/authorizing-an-ssh-key-for-use-with-saml-single-sign-on)

2. Generate new [GPG signing key(s)](https://docs.gitlab.com/ee/user/project/repository/gpg_signed_commits/)
    - Execute
      ```shell
      $ gpg --gen-key
      ```
    - Note your GPG key ID, It begins after the `/` character in the `sec` paragraph after executing:
      ```shell
      $ gpg --list-secret-keys --keyid-format LONG first.last@organisation1.com
      sec   rsa3072/<GPG KEY ID IS HERE> 2022-03-03 [SC]
            5A233D97F169400541080D50D58FC20EB4027CXX
      uid                 [ultimate] first last <first.lastorganisation1.com>
      ssb   rsa3072/560358F2315DB6XX 2022-03-03 [E]
      ```

3. Set up `~/.gitconfig` so that it automatically picks up your git config based on the folder prefix
  - `~/.gitconfig` (we suggest to follow folder paths: `~/Projects/gitlab/my-organisation1` and `~/Projects/github/my-organisation2` for all cloned repositories)
    ```ini
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
    email = first.last@organisation2.com
    name = your-github-username
    signingkey = your GPG key ID
    ```

4. Make sure everything works by cloning some repository using git ssh protocol

## Troubleshooting

- If you get the error error: `gpg failed to sign the data` try running `$ export GPG_TTY=$(tty)` before commiting and add it to your `~/.zshrc` config if it helped

```shell
$ echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
$ source ~/.zshrc
```
