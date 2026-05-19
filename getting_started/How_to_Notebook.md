# Working with Notebooks in GeoLab

Jupyter Notebooks are interactive documents that combine live code, equations, visualizations, and narrative text. They are the primary way most users work in GeoLab.

## What is a Jupyter Notebook?

A notebook is made up of individual **cells** — each cell contains either executable code or formatted text (Markdown). You run cells one at a time, or all at once, and the output appears directly below each cell. This makes notebooks well-suited for exploratory data analysis, documentation, and sharing reproducible workflows.

## Opening and Creating Notebooks

GeoLab uses **JupyterLab**, a browser-based IDE. When you first log in, you'll land on the **Launcher**. From there, click **Python 3 (ipykernel)** under the Notebook heading to open a new, empty notebook.

There are several other ways to reach the Launcher:
- Click the blue **+** button in the top-left corner of the sidebar
- **File → New Launcher**

To open an existing notebook, navigate to it in the **File Browser** (folder icon in the sidebar) and double-click the `.ipynb` file.

See [Navigating GeoLab](./basic_navigation.md) for a full tour of the JupyterLab interface.

## Notebook Interface

When a notebook is open, you'll see three main areas:

- **Menu Bar** — options for File, Edit, View, Run, Kernel, and more
- **Toolbar** — quick-access buttons for saving, adding cells, running cells, and changing cell type
- **Cell area** — the main working area where you write and run code or text

![Notebook Interface](../img/Notebook_interface.png)

## Cell Types

### Code Cells

Code cells run Python (or whichever language your kernel supports). Click a cell to select it, then run it with **Shift + Enter**. Output appears immediately below.

```python
print("Hello, GeoLab!")
```

### Markdown Cells

Markdown cells contain formatted text — useful for explaining your workflow, adding section headers, and documenting results. Change a cell to Markdown using the dropdown in the toolbar, or press **Esc then M** when a cell is selected.

Common Markdown syntax:

| Element | Syntax |
|---|---|
| Heading | `# H1`, `## H2`, `### H3` |
| Bold | `**bold**` |
| Italic | `*italic*` |
| Inline code | `` `code` `` |
| Code block | ```` ```python ... ``` ```` |
| Link | `[text](URL)` |
| Image | `![alt text](path/to/image.png)` |
| Unordered list | `- item` |
| Ordered list | `1. item` |

Double-click a rendered Markdown cell to edit it. Run it again to render.

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Shift + Enter` | Run cell, move to next |
| `Ctrl + Enter` | Run cell, stay in place |
| `Esc` | Enter command mode |
| `Esc then A` | Insert cell above |
| `Esc then B` | Insert cell below |
| `Esc then D, D` | Delete selected cell |
| `Esc then M` | Change cell to Markdown |
| `Esc then Y` | Change cell to code |
| `Ctrl + S` | Save notebook |

For the full list, go to **Help → Show Keyboard Shortcuts**.

## Saving Your Work

Jupyter autosaves periodically, but you should save manually before closing a notebook: **Ctrl + S**, or **File → Save Notebook**.

Notebooks are saved to your `/home/jovyan` directory and will persist between sessions. See [File Systems and Data Storage](./user_storage.md) for storage limits and best practices.

## Exporting Notebooks

To export a notebook to another format (HTML, PDF, etc.):

1. **File → Save and Export Notebook As...**
2. Select your desired format.

PDF export requires LaTeX to be available in your environment.

## Installing Packages

If you need a package that isn't included in the default GeoLab image, install it at the start of your session using `%pip` or `%conda` in a code cell:

```python
%pip install packagename
```

Note that installations are **ephemeral** — they apply only to your current session and will not persist after you log out. See [Environment Management](../advanced_topics/env_mgmt.md) for details on persistent options.

## Best Practices

- **Use Markdown cells** to explain your workflow and document your findings alongside your code.
- **Add section headers** using `#` headings — they enable the Table of Contents sidebar panel and make long notebooks easier to navigate.
- **Keep cells focused** — shorter cells are easier to debug and re-run.
- **Restart and run all before sharing** — use **Kernel → Restart Kernel and Run All Cells** to confirm your notebook runs cleanly from top to bottom.
- **Use Git** to version-control and share your notebooks. See [Using Git](./using_git.md).

## Troubleshooting

**Unexpected errors or stale variable values**
Restart the kernel: **Kernel → Restart Kernel**. This clears all variables and lets you start fresh. Re-run your cells from the top.

**Kernel appears stuck or unresponsive**
Try **Kernel → Interrupt Kernel** first. If that doesn't work, restart the kernel.

**Cluttered or very large cell output**
Right-click the output area and select **Clear Output**, or use **Edit → Clear All Outputs**.

**Wrong environment or missing packages**
Confirm you launched GeoLab with the correct image. See [Launching your Server](./server_launch.md).
