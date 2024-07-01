(devcontainers)=
# Dev Containers

You can use dev containers to build the project if you prefer a streamlined setup experience. This means you can use the same tools and dependencies as the rest of the team, including our Continuous Integration (CI) workflows, without worrying about installing dependencies on your host machine. Additionally, using Dev Containers makes it simple to switch between local or remote development environments, such as GitHub Codespaces.

:::{seealso}
More info on Dev Containers can be found here:

- [Development Containers](https://containers.dev/)
  - An open specification for enriching containers with development specific content and settings
- [Developing inside a Container](https://code.visualstudio.com/docs/remote/containers)
  - Learn how to use Visual Studio Code to develop inside a Docker container
- [GitHub Codespaces overview](https://docs.github.com/en/codespaces/overview)
  - A development environment hosted in the cloud
:::

```{toctree}
:hidden:
:glob:

*
```

## What, Why, How?

Lets briefly explain what dev containers are, why you should use them, and how they work.

:::{hint}
Here we'll assume the use of VS Code, but still applies to alternative tools and services, including  other CLIs, IDEs, etc. such as:

- [Dev Container CLI](https://github.com/devcontainers/cli)
  - A reference implementation for the open specification
- [JetBrains Space | Develop in Dev Environment](https://www.jetbrains.com/help/space/develop-in-a-dev-environment.html)
  - Using Dev Containers with JetBrain based products
- [Supporting tools](https://containers.dev/supporting)
  - List of tools and services supporting the development container specification
:::

### What is a Dev Container?

A dev container is a Docker container that has all the tools and dependencies you need to develop the project. It runs in a self-contained environment and is isolated from other containers and your host machine. This lets you reliably develop for the project anywhere, notably for linux distributions targeted by ROS, regardless of your host machine's operating system.

### Why use a Dev Container?

A dev container provides a common and consistent development environment. It ensures that everyone on the team is using the same tools and dependencies. It also makes it easy to switch between projects because each project can use a different container. This is especially useful if you work on multiple projects that use different versions of the same tools and dependencies, such as different versions of ROS.

### How do Dev Containers work?

When you open the project in VS Code, VS Code checks for the dev container configuration nested within the `.devcontainer` folder under the project's root directory. If it finds one, it can prompt you to reopen the project in a container. If you choose to do so, it launches the container, connects to it, and mounts your project folder inside the container. You can then use VS Code in the container just as you would locally. While setting up the container, VS Code can also attempt to passthrough useful aspects of your local environment, such as git user configurations, X11 sockets, and more.

This is quite similar to earlier tools used to customize and run docker containers for development:

- [rocker | ROS + Docker](https://github.com/osrf/rocker)
  - A tool to run docker containers with overlays and convenient options for things like GUIs etc.
  - Developed by [Open Robotics](https://www.openrobotics.org/)
- [ADE Development Environment](https://ade-cli.readthedocs.io/en/latest/)
  - A modular Docker-based tool to ensure developers have a common, consistent development environment
  - Developed by [Apex.AI](https://www.apex.ai/)

## Prerequisites

To use dev containers, you'll need the following:

- [Docker Engine](https://docs.docker.com/engine/install/) installed and running on the host machine
- [Visual Studio Code](https://code.visualstudio.com/) installed on any remote machine
- [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension installed in VS Code

:::{note}
Alternatively, you could use GitHub Codespaces directly from the project repo, or any other remote host machine:

- [Creating a codespace for a repository](https://docs.github.com/en/codespaces/developing-in-codespaces/creating-a-codespace-for-a-repository?tool=webui)
  - How to create a codespace for repository via GitHub CLI, VS Code, or Web browser
- [Develop on a remote Docker host](https://code.visualstudio.com/remote/advancedcontainers/develop-remote-host)
  - How to connect VS Code to a remote Docker host using SSH tunnels or TCP sockets
:::

## Getting started

Getting started using dev containers is as simple as opening the project in VS Code by either: following the notification prompt to reopen the project in a container, or explicitly opening the command palette (Crtl+Shift+P) and selecting `Remote-Containers: Reopen in Container`. This will create a new container, install any extensions specified in the project's default `.devcontainer/devcontainer.json` config file, and mount the project's root directory as the workspace folder. Once the container is created, VS Code will connect to it and you can start developing.

:::{tip}
Clicking the `Starting Dev Container (show log)` notification in VS Code allows you to observe in live time how the sausage is made, while typing `Dev Containers: Show Log` into the command palette will list all the available commands to review and revisit these log files later.
:::

While waiting for the initial setup, feel free to stretch your legs, grab a coffee, or continue to read the following guides to learn more about creating and using dev containers, or how to visualize and leverage graphical user interfaces from a headless development environment.

- **[](devcontainer_guide.md)**
  - How to develop Nav2 using dev containers and supporting tools
- **[](visualization_guide.md)**
  - How to visualize Nav2 using local or remote headless containers

## Security

:::{caution}
Ensure you trust the authors and contents of workspaces before launching derived dev containers.
:::

A word of caution when using dev containers: they are powerful tools, but can be a security concern, as the capability of arbitrary code execution facilitated by IDE extensions to enable such automation and convenience remains inherently dual use. Before launching a dev container, ensure you trust the workspaces and authors. For example, when reviewing a pull request, verify patches remain benign and do not introduce any malicious code. Although such vigilance is merited whenever compiling and running patched code, using containers with either elevated privileges or filesystem access renders this diligence even more prudent.

:::{seealso}
More info on trusting workspaces and extensions in general can be found here:

- [Workspace Trust](https://code.visualstudio.com/docs/editor/workspace-trust)
  - VS Code user guid on trusting and configure workspaces
:::
