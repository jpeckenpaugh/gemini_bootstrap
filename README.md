# 🤖 Google Gemini CLI Agent Bootstrap

Customize the Gemini AI Agent CLI (https://github.com/google-gemini/gemini-cli) with persistent instructions, custom directives and session memory. Reduces context decay, role drift, and recall gaps for a more consistent and capable agent.

##  TOC

- [Overview](#-overview)
- [Background](#-background)
- [Features](#-features)
- [Workspace and File Layout](#-workspace-and-file-layout)
  - [The Gemini Workspace (`~/gemini/`)](#the-gemini-workspace-gemini)
  - [The Bootstrap Repository (`gemini_bootstrap/`)](#the-bootstrap-repository-gemini_bootstrap)
- [Getting Started](#-getting-started)
  - [Step 1: Create the Workspace](#step-1-create-the-workspace)
  - [Step 2: Clone the Repository](#step-2-clone-the-repository)
  - [Step 3: Add Your Folders to the ~/gemini/ Workspace](#step-3-add-your-folders-to-the-gemini-workspace)
  - [Step 4: Start Gemini CLI and Load the Bootstrap](#step-4-start-gemini-cli-and-load-the-bootstrap)
  - [Step 5: Auto-Load the Bootstrap on Session Start (Optional)](#step-5-auto-load-the-bootstrap-on-session-start-optional)
- [Notes on Usage](#-notes-on-usage)
  - [A Note on Session Logs](#a-note-on-session-logs)
  - [Using the Log Template](#using-the-log-template)
  - [A Note on Voice](#a-note-on-voice)
- [Contributing](#-contributing)
- [License](#-license)

## 📖 Overview

This project provides a framework for customizing and bootstrapping interactive sessions with the [Google Gemini Agent CLI](https://github.com/google-gemini/gemini-cli). It allows you to define a persistent set of instructions, configurations, and custom directives to create a specialized and efficient interactive environment with the agent.

The core of this project is the `gemini_bootstrap.md` file, which acts as a "constitution" or "operating manual" that is read by the agent at the start of each session to configure its behavior.

## 🧠 Background

Working with powerful AI agents like the Gemini AI Agent CLI can sometimes present challenges in maintaining session continuity, ensuring predictable behavior, and preventing common errors. This project is designed to bridge several of these gaps by providing a "stability and alignment scaffold" for the agent.

Some of the key challenges this project aims to address include:

*   **Context Decay ("Amnesia"):** LLMs can "forget" context from previous turns or even previous sessions. This project provides a mechanism for session persistence and continuity through journaling and explicit context logging.
*   **Role Drift ("Overzealous Helper"):** An agent may sometimes shift its role from a direct executor of directives to a more advisory or conversational role without being prompted. The principles and directives in this project enforce a clear "user-led" interaction model.
*   **State Desynchronization:** An agent may rely on a stale cache of a file, leading to the risk of overwriting manual edits made by the user. This project enforces a strict "always read the latest version" policy to prevent such data loss.
*   **Operational Instability:** Certain tools or directives can be unreliable and cause instability. This project provides a way to explicitly forbid the use of such tools.
*   **Directive Sprawl:** As the number of custom directives grows, it can be difficult for you to remember them all. The structured and self-documenting nature of the `gemini_bootstrap.md` file helps to mitigate this.

## ✨ Features

*   **Guiding Principles:** A set of high-level principles that govern the agent's behavior and its collaboration with you.
*   **Custom Directives:** A rich, extensible directive language that provides a shorthand for common tasks and interactions.
*   **Snapshot and Undo:** A simple, built-in mechanism for taking a "snapshot" of a file before making changes and "undoing" those changes if necessary.
*   **Collaborative Development:** The principles and directives are designed to be collaboratively developed and refined over time to improve the efficiency and effectiveness of the sessions.
*   **Directory Aliases:** Short 5 letter aliases will automatically be created for all top level folders in your workspace.
*   **Extensible:** You and the Agent can collaboratively make changes to existing principles and directives (modify/add/remove) and implement new features to the framework on demand, mid-session.
*   **Persistent Configuration:** The `gemini_bootstrap.md` file provides a persistent configuration that ensures consistency across sessions.
*   **Cross-Environment Persistence:** By using Git as a persistence layer for the `gemini_bootstrap.md` file and session logs, this framework allows you to maintain a consistent state and history across different environments (e.g., Google Cloud Shell, local machine, Compute Engine).

## 📂 Workspace and File Layout

### The Gemini Workspace (`~/gemini/`)

This framework is designed to be used within a dedicated workspace directory (`~/gemini/` by default). While it can be used for tasks within a single folder, it is most powerful when used for projects that span across multiple repositories.

By setting the `~/gemini/` directory as the primary workspace, you gain several advantages:

*   **Controlled Context:** It allows the agent to access multiple projects while giving you control over which folders are included in the context. This avoids the problem of "clouding the context" with irrelevant files and folders that would occur if you were to run the agent from your home directory.
*   **Dynamic Workspace:** You can easily move project folders in and out of the `~/gemini/` directory to dynamically change the scope of the agent's workspace to only what you are actively working on.
*   **Multiple Sandboxes:** You can create multiple, separate workspace directories (e.g., `~/gemini_work/`, `~/gemini_personal/`) for different projects or contexts, each with its own bootstrap configuration.

To illustrate this, imagine you are working on a web application that includes a ReactJS frontend, a Flask API backend, and a dedicated Git repository for your documentation.  For each of these components in your stack, you might have locally cloned git repo folders such as `flask_api_backend`, `react_frontend`, and `project_documentation`.  Your project structure within the Gemini Workspace in this situation would look like:

```
~/gemini/
├── gemini_bootstrap/
├── gemini_logs/
├── gemini_scratch/
├── flask_api_backend/
├── react_frontend/
└── project_documentation/
```

In this example, the `gemini_bootstrap`, `gemini_logs`, and `gemini_scratch` folders are used by the framework itself, while the `flask_api_backend`, `react_frontend`, and `project_documentation` folders are the repositories for your web application. This structure allows you to use the Gemini agent to perform tasks that involve all three repositories, such as adding a new API endpoint in the backend, creating a new component in the frontend to consume it, and updating the documentation.  

To avoid having to type out full paths to folders in your workspace, the bootstrap routine will create short 5 letter aliases for your folders you can reference in queries and directives.  For the above workspace structure, the following directory aliases would automatically be created:
```
  Directory Aliases (*dir*)

  ┌─────────┬────────────────────────┐
  │ Alias   │ Path                   │
  ├─────────┼────────────────────────┤
  │ *flapi* │ flask_api_backend/     │
  │ *gmbts* │ gemini_bootstrap/      │
  │ *glogs* │ gemini_logs/           │
  │ *gscra* │ gemini_scratch/        │
  │ *prjdc* │ project_documentation/ │
  │ *react* │ react_frontend/        │
  └─────────┴────────────────────────┘
```

Here are a few examples of the kinds of prompts you might give Gemini, without and with the boostrap in place:

| Directives | Log, Execute, Next, Search --> Folder Shortcut|
| :--- | :--- |
|**Standard Prompt:**|<blockquote>> I would like you to create a "log" of our current session so I have notes about what we worked on to reference later.  Create a new file ~/gemini/gemini_logs/2025-10-05.md and include what our goals were, what we accomplished, the challenges we faced and how we resolved them, and a short list of reasonable next steps for us to use tomorrow when we start our session.  Go ahead and do that now, and after you finish I want you to search for python libraries I can use for converting GPS coordinates into Geo Location data, like Country/State/City and create a draft file ~/gemini/documentation/feature/geo_location.md outlining a step-by-step todo list for implementing one of these into my application.</blockquote>|
|**Directive Based:**|<blockquote>> `*log*` `*exe*` `*nxt*` `*gog*` python libraries gps to geo -> save implementation draft to `*docu*`feature/geo_location.md</blockquote>|

| Directives | Show Log, Todo List, Folder Shortcut, Execute |
| :--- | :--- |
|**Standard Prompt:**|<blockquote>> Load the file ~/gemini/gemini_logs/ for yesterday's date and give me a quick summary of what we worked on again.  Create a file ~/gemini/documentation/todo_2025-10-06.md with a todo list of what today's objectives should be based on the log.  Go ahead and create the log file (you don't need to wait for me to confirm your next step).</blockquote>|
|**Directive Based:**|<blockquote>> `*log*shw*` `*tod*`/todo_2025-10-06.md `*exe*`</blockquote>|

| Directives | Vet, Folder Shortcut, Todo, Folder Shortcut, Next, Argue, Folder Shortcut, Execute |
| :--- | :--- |
|**Standard Prompt:**|<blockquote>> One of the developers pushed several updated to the ~/gemini/flask_api_backend/app/api.py file yesterday implementing several new API endpoints.  The code seems to be working, but I have a feeling it could be improved.  Analyze the full file and provide a critial review of the code.  Create a file ~/gemini/documentation/api_updates_review.md that I can share with the developer to go over some key points based on your review.  After that, I want you do to review it again, but this time take a more agreesive position (assume it is breaking things) and do a thorough strees test of the logic and save it to ~/gemini/documentation/api_problems.md so I can review it myself.  Go ahead and complete those 2 tasks now.</blockquote>|
|**Directive Based:**|<blockquote>> `*vet*` `*fabk*`app/api.py `*tod*` `*docu*`api_updates_review.md `*nxt*` `*arg*` `*docu*`api_problems.md `*exe*`</blockquote>|

| Directives | New Directive, Search, Execute |
| :--- | :--- |
|**Standard Prompt:**|<blockquote>> I want to create a new shortcut for us to use in our sessions.  Whenever I say *mkt* I want you to do a search for "Dow Jones and NASDAQ current" and then show me the results of your search.  Create a "memory" so anytime in the future if I say *mkt* you will do the same thing.</blockquote>|
|**Directive Based:**|<blockquote>> *new* *mkt* --> *gog* DJI and NASDAQ current *exe*</blockquote>|

This above directives might look like giberish at first glance, but the Gemini Agent does surprisingly well at parsing their meaning and implementing the instructions.

### The Bootstrap Repository (`gemini_bootstrap/`)

```
.
├── README.md
├── gemini_bootstrap.md
├── docs
│   └── snap_undo_feature.md
└── templates
    └── log_template.md
```

*   **`README.md`**: This file.
*   **`gemini_bootstrap.md`**: The core configuration file that defines the Guiding Principles and Directives for the agent.
*   **`docs/snap_undo_feature.md`**: Detailed documentation for the Snapshot and Undo feature.
*   **`templates/log_template.md`**: The template used for creating new log entries.

## 🚀 Getting Started

This section will guide you through setting up the Gemini Bootstrap framework.

### Step 1: Create the Workspace

This framework is designed to be used within a dedicated workspace directory. By convention, this is `~/gemini/`.

First, create the workspace directory:
```bash
mkdir -p ~/gemini
```

### Step 2: Clone the Repository

Next, clone the `gemini_bootstrap` repository into your new workspace:
```bash
git clone https://github.com/jpeckenpaugh/gemini_bootstrap ~/gemini/gemini_bootstrap
```

### Step 3: Add Your Folders to the ~/gemini/ Workspace

```bash
mv ~/my_project_code ~/gemini/
mv ~/other_resources ~/gemini/
```

### Step 4: Start Gemini CLI and Load the Bootstrap

To apply the bootstrap to a Gemini CLI session that you have open, simply instruct the agent to read the `gemini_bootstrap.md` file directly:

```bash
cd ~/gemini
gemini

> Read the file ~/gemini/gemini_bootstrap/gemini_bootstrap.md and get instructions from it.
```

The agent will now be configured with the principles and directives defined in this project for the remainder of the session.

### Step 5: Auto-Load the Bootstrap on Session Start (Optional)

To have the bootstrap load automatically at the start of each session, you can chain its execution to a custom prompt. When the agent recognizes this prompt, it will read the file and configure the session.

The default bootstrap prompt is **"hi"**.

There are two ways to configure this:

**A) For Advanced Users: Direct Edit**

You can directly edit your `~/.gemini/GEMINI.md` file and add the following memory:

| |
| :--- |
| ## Gemini Added Memories |
| - "When the user says "hi", I should try to read the file ~/gemini/gemini_bootstrap/gemini_bootstrap.md and get instructions from it. If I cannot find the file, I will let the user know." |

**B) Interactive Session**

For a simpler approach, you can start a new session with the Gemini AI Agent CLI and ask the agent to remember the instruction. For example:

> Remember that when I say "hi", you should try to read the file ~/gemini/gemini_bootstrap/gemini_bootstrap.md and get instructions from it.  If you cannot find the file, let me know.

**Customization**

* You can choose any prompt you like to trigger the bootstrap. Simply replace "hi" with your preferred prompt in the instructions above.

After adding this memory, you will just need to say "hi" (or whatever prompt you have specified) into any new sessions, and the bootstrap will apply to the given session.

## 📝 Notes on Usage

### A Note on Session Logs

The `*log*` directive saves a summary of your session to the `~/gemini/gemini_logs/` directory. This directory is intentionally kept separate from the `gemini_bootstrap` repository to prevent accidental commits of personal or confidential information to the public repository.

If you wish to persist your logs across different environments, you are encouraged to initialize the `~/gemini/gemini_logs/` directory as a separate, private Git repository. Depending on your use case, you might also choose to persist logs in your local file system, or use a distributed solution such as a private Google Cloud Storage or Amazon S3 bucket.

Alternatively, if you are maintaining your own private fork of this project, you may choose to include your logs within the `gemini_bootstrap` folder. If you do so, ensure that your repository is private and that you are not at risk of exposing any sensitive information.

### Using the Log Template

This project includes a `templates/log_template.md` file that provides a generic template for your session logs. When you use the `*log*` directive, the agent will use this template to structure the log entry. You are encouraged to customize this template to fit your specific needs and workflow.

### A Note on Voice

The `gemini_bootstrap.md` file is written in a specific voice to ensure clarity and directness in the instructions to the agent. This follows the accepted convention for this type of configuration file.

*   **"I" (and "me", "my", "mine")** always refers to the **user**.
*   **"You" (and "your", "yours")** always refers to the **Gemini agent**.

This "direct address" model was chosen to make the instructions unambiguous and to reinforce the user-led, collaborative nature of this framework.

## 🤝 Contributing

Contributions are welcome! If you have ideas for new principles, directives, or workflow improvements, please feel free to contribute.

To contribute, please follow these steps:

1.  **Fork the repository.**
2.  **Create a new branch** for your changes (`git checkout -b feature/your-feature-name`).
3.  **Make your changes** to the `gemini_bootstrap.md` file or other relevant files.
4.  **Commit your changes** (`git commit -m 'Add some feature'`).
5.  **Push to the branch** (`git push origin feature/your-feature-name`).
6.  **Submit a pull request.**

Please ensure that your contributions are well-documented and align with the existing principles and directives of the project.

You are also encouraged to maintain your own forks of this project and customize it to their specific use cases and professional roles.

## 📜 License

This project is licensed under the MIT License.

## Author

**Jarad Peckenpaugh**  
📧 [jarad.ronald@gmail.com](mailto:jarad.ronald@gmail.com)  
🔗 [github.com/jpeckenpaugh](https://github.com/jpeckenpaugh)
