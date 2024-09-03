# .gitconfig-setup

How to prepare your git environment to work:
- with different auth `ssh` and signing `gpg` keys
- with different git hosts (e.g. GitHub, GitLab) and organization (private account, company account) 
- with automatic key selection based on the folder path and ssh host

Steps:
1. Generate new [SSH private key(s)](https://docs.gitlab.com/ee/user/ssh.html#generate-an-ssh-key-pair)
    - Generating a private key with a **passphrase** is a requirement on Mac :warning:
      ```shell
      $ ssh-keygen -t ED25519 -C "your@github.email" -f ~/.ssh/id_org1_github
      $ pbcopy ~/.ssh/id_org1_github.pub
      $ open https://github.com/settings/keys
  
      $ ssh-keygen -t ED25519 -C "your@gitlab.email" -f ~/.ssh/id_org3_gitlab
      $ pbcopy ~/.ssh/id_org3_gitlab.pub
      $ open https://gitlab.com/-/profile/keys
      ```
    - [Make keys "persistent"](https://unix.stackexchange.com/a/560404/171941) automatically loaded after Mac reboot
    - Update `~/.ssh/config` file:
      ```config
      Host *
          UseKeychain yes
          AddKeysToAgent yes
          IgnoreUnknown UseKeychain
      
      Host org1-github
          user = git
          HostName = github.com
          identityfile = ~/.ssh/id_org1_github
          identitiesonly yes           

      Host org3-gitlab
          HostName = gitlab.com
          user = git
          identityfile = ~/.ssh/id_org3_gitlab
          identitiesonly yes
      ```
    - For GitHub make sure you authorized your ssh key with [your organization via SSO](https://docs.github.com/en/enterprise-cloud@latest/authentication/authenticating-with-saml-single-sign-on/authorizing-an-ssh-key-for-use-with-saml-single-sign-on)

1. Generate new [GPG signing key(s)](https://docs.gitlab.com/ee/user/project/repository/gpg_signed_commits/)
    - Execute
      ```shell
      $ gpg --gen-key
      ```
    - Note your GPG key ID, It begins after the `/` character in the `sec` paragraph after executing:
      ```shell
      $ gpg --list-secret-keys --keyid-format LONG first.last@organisation1.com
      sec   rsa3072/<GPG KEY ID IS HERE> 2022-03-03 [SC]
            5A233D97F169400541080D50D58FC20EB4027CXX
      uid                 [ultimate] first last <first.last@organisation1.com>
      ssb   rsa3072/560358F2315DB6XX 2022-03-03 [E]
      ```

1. Set up `~/.gitconfig` so that it automatically picks up your git config based on the folder prefix
  - We suggest to follow folder paths for all git repositories
    ```shell
    ~/Projects
    ├── github
    │   ├── my-organisation1
    │   │   ├── repo-1
    │   │   └── ...
    │   └── my-organisation2
    │       ├── repo-2
    │       └── ...
    ├── gitlab
    │   └── my-organisation3
    │       ├── repo-3
    │       └── ...
    ├── bitbucket
    │   └── my-organisation4
    │       ├── repo-4
    │       └── ...
    ...
    ```
  - `~/.gitconfig` 
    ```ini
    [commit]
    gpgsign = true

    [includeIf "gitdir:~/Projects/github/my-organisation1/"]
    path = ~/.gitconfig-github-my-organisation1

    [includeIf "gitdir:~/Projects/gitlab/my-organisation3/"]
    path = ~/.gitconfig-gitlab-my-organisation3
    ```

  - `~/.gitconfig-github-my-organisation1`
    ```ini
    [user]
    email = first.last@organisation1.com
    name = your-github-username
    signingkey = your GPG key ID

    [url "ssh://git@org1-github/umg/"]
    insteadOf = git@github.com:org1/
    ```

  - `~/.gitconfig-gitlab-my-organisation3`
    ```ini
    [user]
    email = first.last@organisation3.com
    name = your-gitlab-username
    signingkey = your GPG key ID

    [url "ssh://git@org3-gitlab/umg/"]
    insteadOf = git@gitlab.com:org3/
    ```

4. Make sure everything works by cloning some repositories using git ssh protocol

## Troubleshooting

1. Read git config variables from respective paths
    ```shell
    $ cd ~/Projects/github/my-organisation1/repo-1 on main
    $ git config --show-origin --get user.name
    
    file:~/.gitconfig-github-my-organisation1 your-github-username
    ```
    
    ```shell
    $ cd ~/Projects/gitlab/my-organisation3/repo-3 on main
    $ git config --show-origin --get user.name

    file:~/.gitconfig-gitlab-my-organisation3 your-gitlab-username
    ```
1.1 If you made changes to repos with an existing .git/config, then you need to remove it
```
rm .git/config
git init
# Check if change is there
git config --show-origin --get user.name
```
    
1. If you get the error: `gpg failed to sign the data` try running `$ export GPG_TTY=$(tty)` before committing and add it to your `~/.zshrc` config if it helped
    ```shell
    $ echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
    $ source ~/.zshrc
    ```

1. Check ssh auth log what is happening under the hood
    ```shell
    $ ssh -Tv git@github.com
    ```
