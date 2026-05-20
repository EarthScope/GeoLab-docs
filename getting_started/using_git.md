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

### 1. Tell git who you are
In the terminal, run the following commands:
```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```
:::{note}
Using the `--global` flag will apply these settings to every git project in your GeoLab account. If you have multiple git identities and wish to use a different identity for a specific project, run the commands above inside the specific project folder, without the `--global` flag.
:::

### 2. Authenticate
Depending on how your git accounts are configured, you may need to provide authentication when syncing code to/from your remote repository.
Option 1 - enter your password every time you make a push to your remote. 
Option 2 - set up an ssh key (instructions coming soon)
<!-- todo: add ssh key instructions -->

## Best Practices
- Avoid nested repository cloning (cloning a repository inside another repository), and keep all your clones in a single directory — `/home/jovyan/` is a good choice.