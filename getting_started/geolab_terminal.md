# Using the Terminal in GeoLab

The terminal gives you access to a command-line shell for organizing files, installing software, cloning repositories, and running scripts. GeoLab uses `bash` by default.

## How to Open a Terminal
- File &rarr; New &rarr; Terminal
- Open a new [Launcher](./basic_navigation.md/#main-panel) and click the Terminal button

## Managing Terminals
You can have many terminals open in GeoLab at once. This is helpful when you need to type a command without disrupting another running process.

Clicking the **X** on a terminal tab closes the tab but does not stop the process. Running terminals can be retrieved from the [Usage Browser](./basic_navigation.md#usage-browser)


## User `jovyan` and Root Privileges
Your default path `:~` in the Terminal window refers to `/home/jovyan`. This is prefaced by the user id, or `jovyan@jupyter-your-unique-account-identification-number` which is tied to your email or Google/Cilogon profile. Due to the NSF NGF's authentication implementation, there is no convenient way to shorten this path prefix in the terminal display. 

`Jovyan` users do not have administrator or root permissions, and therefore cannot use the `sudo` command to override system commands.