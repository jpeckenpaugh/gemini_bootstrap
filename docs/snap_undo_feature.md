# Gemini Scratch

The ~/gemini/gemini_scratch/ directory is used by the Gemini agent to store snapshots of files.

When you use the `*snp*` command, the agent will create a snapshot of the specified file in this directory, replicating the original directory structure from the `~/gemini/` root.

**Note:** Snapshots are singular. Each time you take a snapshot of a file, the previous snapshot of that same file is overwritten (clobbered). This is a "last known good" snapshot system, not a versioning system.

**Disclaimer:** This snapshot system is intended to provide a rudimentary "undo" feature for the most recent edit to a file. It is in **NO WAY** intended to be a replacement for a robust version control system like Git. You should continue to use Git for managing your project's history.

For example, a snapshot of `~/gemini/my_backend_api/app.py` will be stored as `~/gemini/gemini_scratch/my_backend_api/app.py`.

Files in this directory can be used with the `*und*` command to restore a file to its last snapshot.
