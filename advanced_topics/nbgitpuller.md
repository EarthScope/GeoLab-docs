# Sharing Notebooks with nbgitpuller

nbgitpuller is a JupyterHub extension that lets you create a single shareable link that automatically pulls a Git repository into a recipient's hub environment and opens a specific file — no terminal or git commands required on the user's end. It is particularly useful for educators and workshop organizers who want to distribute notebooks to participants.

When a user clicks an nbgitpuller link, GeoLab will:
1. Clone the repository into their `/home/jovyan` directory if it doesn't exist, or sync it if it does
2. Open the specified notebook or file directly

Importantly, nbgitpuller uses a **one-way merge** strategy: it pulls changes from the remote repository without overwriting any local edits the user has made. This means participants can work on a notebook and receive upstream updates without losing their work.

## Generating a link

The easiest way to create an nbgitpuller link is with the [nbgitpuller Link Generator for GeoLab](https://nbgitpuller.readthedocs.io/en/latest/link.html?hub=https://geolab.earthscope.cloud). You will need:

- **JupyterHub URL** — the GeoLab hub URL
- **Repository URL** — the public GitHub repository containing your notebooks
- **Branch** — the branch to pull from (usually `main`)
- **File to open** — the path to the notebook or file to open on launch (optional)

## Further reading

See the [nbgitpuller documentation](https://nbgitpuller.readthedocs.io) for full details on link options, the merge strategy, and troubleshooting.
