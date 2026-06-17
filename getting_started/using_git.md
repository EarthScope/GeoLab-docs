# Using Git with GeoLab

Git is a version control system installed in all GeoLab environments. You can clone repositories from GitHub or any other platform directly into your GeoLab environment.

You are responsible for managing the repositories you clone. Per the [Acceptable Use Policy](../welcome/geolab_AUP.md), GeoLab administrators are not responsible for issues arising from repository management.

## Learning Git

There are many, _many_ resources for learning to use git. While GeoLab does have an integrated git dashboard on the sidebar, we recommend learning the git CLI (command line interface) so your learning is portable to any future compute environments you might find yourself in. Once you understand git on the command line, using any version of a git GUI is an easy transition. 

Recommended resources:
- [Why Git? Illustrated](https://openscapes.org/blog/2022-05-27-github-illustrated-series/)
- [Happy Git](https://happygitwithr.com/) — While targeted at R users, very little of this guide is R-specific; it's an excellent resource for all git learners
- [Free Code Camp: Learn Git in Under 10 Minutes](https://www.freecodecamp.org/news/learn-the-basics-of-git-in-under-10-minutes-da548267cc91/)

## Git in GeoLab
Once you've logged into GeoLab, you can use git from the command line in the terminal. 
There is also a visual git GUI extension available by clicking the git icon ![Git icon](../img/git_icon.png) on the left sidebar. 

## Connecting Accounts
If you have a github or gitlab account and wish to sync projects to it from GeoLab, you will need to run a few configuration steps. 
**Note, git account setup is not required to clone a public repository.** 

### 1. Tell git who you are
In the terminal, run the following commands:
```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```
You'll only need to do this once. Your configs will be stored to a file if you ever need to modify them.

:::{note}
Using the `--global` flag will apply these settings to every git repository in your GeoLab account. If you have multiple git identities and wish to use a different identity for a specific project, run the commands above _inside the specific project folder_, without the `--global` flag.
:::

### 2. Authenticate

An SSH key lets you authenticate with your remote repository securely and without entering your password each time. The steps to set up your SSH key are as follows:

1. **Generate SSH Key**

    In the GeoLab terminal, run:

    ```shell
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```

    When prompted, press `Enter` to accept the default file location. The default location is in the `/home/jovyan/` directory, which persists between GeoLab logins. You can optionally set a passphrase or leave it empty for no passphrase.

2. **Copy your public key**

    Display your public key using the following command in the GeoLab terminal:

    ```shell
    cat ~/.ssh/id_ed25519.pub
    ```
    Copy the entire output to your clipboard.

3. **Add the key to your Git account**

    *For GitHub:*

    1. Go to Settings → SSH and GPG keys → New SSH Key
    2. Type in a title for the key, paste your key and click Add SSH key.

    *For GitLab:*

    1. Go to Preferences → Access → SSH Keys → Add new key
    2. Paste your key, type in the title, set an expiration date for the key (optional) and click Add key.

4. **Test the connection**

    *For GitHub:*

    ```shell
    ssh -T git@github.com
    ```
    Expected output:

    Hi `your_username` ! You've successfully authenticated, but GitHub does not provide shell access.

    *For GitLab:*

    ```shell
    ssh -T git@gitlab.com
    ```

    Expected output:

    Welcome to GitLab, `@your_username`!

## Best Practices
- Avoid nested repository cloning (cloning a repository inside another repository), and keep all your clones in a single directory — `/home/jovyan/` is a good choice.