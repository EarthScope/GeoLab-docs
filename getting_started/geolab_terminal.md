# Using the Terminal in GeoLab

Users will often need to use an interactive command-line shell type to organize files on their home directory, install software that might not be included in the base GeoLab image, clone repositories into their environment using git, or develop a shell-scripting data analysis workflow.

GeoLab provides easy access to the Terminal window to accomplish all of these tasks. By default, GeoLab uses the Bourne Again Shell, `bash`. 

## How to Open a Terminal
There are several ways to open a new terminal:
- File &rarr; New &rarr; Terminal
- Open a new [Launcher](./basic_navigation.md/#main-panel) and click the Terminal button

## Managing Terminals
You can have many terminals open in GeoLab at once. This is helpful when you need to type a command without disrupting another running process.

Please note that clicking the **X** on the terminal tab simply closes the tab; it does not kill the terminal process. Running terminals can be retrieved from the [Usage Browser](./basic_navigation.md#usage-usage-browser)


## User `jovyan` and Root Privileges
Your default path `:~` in the Terminal window refers to `/home/jovyan`. This is prefaced by the user id, or `jovyan@jupyter-your-unique-account-identification-number` which is tied to your email or Google/Cilogon profile. Due to EarthScope's authentication implementation, there is no convenient way to shorten this path prefix in the terminal display. 

`Jovyan` users do not have administrator or root permissions, and therefore cannot use the `sudo` command to override system commands.