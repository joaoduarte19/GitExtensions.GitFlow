# GitExtensions.GitFlow

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![.NET](https://img.shields.io/badge/.NET-10.0--windows-512BD4)](https://dotnet.microsoft.com/)
[![Git Extensions](https://img.shields.io/badge/Git%20Extensions-7.0-F05133)](https://github.com/gitextensions/gitextensions)

A [Git Extensions](https://github.com/gitextensions/gitextensions) plugin that brings the
**[git-flow](https://nvie.com/posts/a-successful-git-branching-model/)** branching model into the
Git Extensions GUI. Instead of remembering the `git flow` command line, you drive feature, release,
hotfix, bugfix and support branches from a single dialog.

> This is a standalone, NuGet-distributed revival of the GitFlow plugin that used to ship with Git
> Extensions. The original built-in plugin was removed in Git Extensions 7.0 by
> [gitextensions/gitextensions#12894](https://github.com/gitextensions/gitextensions/pull/12894)
> (_"remove obsolete GitFlow plugin"_, merged March 2026); this project brings it back, rebuilt for
> the Git Extensions 7.0 plugin architecture (.NET 10).

<!-- Add a screenshot of the dialog here once available:
![GitFlow dialog](docs/screenshot.png)
-->

## Features

- **Initialize** git-flow in a repository with default branch settings (`git flow init -d`).
- **Start** a new branch of any git-flow type:
  - `feature`, `bugfix`, `hotfix`, `release`, `support`
  - optionally based on a chosen local branch.
- **Finish** a branch, with optional **push after finish** (`-p`) and **squash** (`-S`).
- **Publish** a branch to a remote.
- **Pull** a branch from a selected remote.
- Detects the current `HEAD` and automatically pre-selects its branch type.
- Reports the executed `git flow` command and its output (success/error) right in the dialog.
- Signals Git Extensions to refresh after operations so the commit graph stays in sync.

## Requirements

- **Windows** with **[Git Extensions](https://github.com/gitextensions/gitextensions) 7.0** (or any
  `7.x` release).
- **[git-flow](https://github.com/petervanderdoes/gitflow-avh)** installed and available on your
  `PATH`. The plugin shells out to `git flow …`, so the git-flow extension (the AVH edition is
  recommended) must be installed separately. Git for Windows ships with it in many setups; otherwise
  install it manually.
- The .NET 10 Windows runtime is provided by the Git Extensions host — you don't need to install it
  separately to *use* the plugin.

## Installation

### Via the Plugin Manager (recommended)

Install it through the
[Git Extensions Plugin Manager](https://github.com/gitextensions/gitextensions.pluginmanager), which
fetches the package and places the assemblies in the correct folder for you.

### Manual installation

1. Build the project (see [Building from source](#building-from-source)) or grab
   `GitExtensions.GitFlow.dll` from a release.
2. Copy `GitExtensions.GitFlow.dll` (and its `.pdb`, if you want symbols) into the folder Git
   Extensions loads user plugins from. The Plugin Manager is the supported way to find/manage this
   location.
3. Restart Git Extensions.

## Usage

1. Open a repository in Git Extensions.
2. Launch the plugin from **Plugins → GitFlow**.
3. If the repository hasn't been initialized for git-flow yet, click **Init** — this runs
   `git flow init -d` to set up the standard branch layout.
4. **Start a branch:** pick a type (`feature`, `bugfix`, `hotfix`, `release`, `support`), enter a
   name, and optionally tick *Based on* to branch off a specific local branch.
5. **Manage existing branches:** choose a type to list its branches, then:
   - **Finish** – completes the branch (available for every type except `support`); enable
     *push after finish* (hotfix/release) and/or *squash* as needed.
   - **Publish** – pushes the branch to a remote.
   - **Pull** – pulls the branch from a selected remote.

The executed command and its result are shown in the dialog, and Git Extensions refreshes
automatically when an operation changes the repository.

## Building from source

### Prerequisites

- **.NET 10 SDK**
- **Visual Studio 2022/2026** with the **.NET Desktop (Windows Forms)** workload, or the `dotnet` CLI
- A local **Git Extensions 7.0** installation (the build references its assemblies)

### Configure the Git Extensions path

The project references Git Extensions assemblies (`GitExtensions.Extensibility.dll`, `GitCommands.dll`,
`GitExtUtils.dll`, `ResourceManager.dll`, …) via the `GitExtensionsPath` MSBuild property. This
property is **not** committed (it lives in the git-ignored `*.csproj.user`), so a fresh clone has to
provide it. Either:

- create `src/GitExtensions.GitFlow/GitExtensions.GitFlow.csproj.user`:

  ```xml
  <Project>
    <PropertyGroup>
      <GitExtensionsPath>C:\Program Files\GitExtensions</GitExtensionsPath>
      <!-- Optional: enables F5 debugging by launching the host -->
      <GitExtensionsExecutablePath>C:\Program Files\GitExtensions\GitExtensions.exe</GitExtensionsExecutablePath>
    </PropertyGroup>
  </Project>
  ```

- or pass it on the command line: `-p:GitExtensionsPath="C:\Program Files\GitExtensions"`.

### Build

```bash
dotnet build src/GitExtensions.GitFlow/GitExtensions.GitFlow.csproj -c Release -p:GitExtensionsPath="C:\Program Files\GitExtensions"
```

The build produces only the plugin DLL — the host supplies the shared dependencies at runtime.

### Package (NuGet)

```bash
dotnet pack src/GitExtensions.GitFlow/GitExtensions.GitFlow.csproj -c Release
```

This uses [`GitExtensions.GitFlow.nuspec`](src/GitExtensions.GitFlow/GitExtensions.GitFlow.nuspec) to
produce a `.nupkg` whose `GitExtensions.GitFlow.dll`/`.pdb` are packed under `lib`, with a dependency
on the virtual `GitExtensions.Extensibility` package (`[7.0,8.0)`).

### Debugging

With `GitExtensionsExecutablePath` set, pressing **F5** launches the installed `GitExtensions.exe`
so you can debug the plugin against a real host.

## Project structure

| Path | Purpose |
| --- | --- |
| [`src/GitExtensions.GitFlow/GitFlowPlugin.cs`](src/GitExtensions.GitFlow/GitFlowPlugin.cs) | Plugin entry point — the `[Export(typeof(IGitPlugin))]` registration. |
| [`src/GitExtensions.GitFlow/GitFlowForm.cs`](src/GitExtensions.GitFlow/GitFlowForm.cs) | The dialog and all git-flow command logic. |
| [`src/GitExtensions.GitFlow/GitExtensions.GitFlow.csproj`](src/GitExtensions.GitFlow/GitExtensions.GitFlow.csproj) | Build configuration and Git Extensions references. |
| [`src/GitExtensions.GitFlow/GitExtensions.GitFlow.nuspec`](src/GitExtensions.GitFlow/GitExtensions.GitFlow.nuspec) | NuGet packaging manifest. |
| [`Directory.Packages.props`](Directory.Packages.props) | Central NuGet package version management. |

### Plugin metadata

| | |
| --- | --- |
| **Name** | GitFlow |
| **Id (GUID)** | `83E1F3F1-B502-4BFB-97D9-7EF108252401` |
| **Target framework** | `net10.0-windows` |
| **Git Extensions API** | `GitExtensions.Extensibility [7.0, 8.0)` |
| **Version** | 1.0.1 |

## How it works

The plugin implements `IGitPlugin` / `IGitPluginForRepository`. When invoked it opens `GitFlowForm`,
which builds and runs `git flow …` commands through the Git Extensions Git executable wrapper and
surfaces the output back in the dialog. It does not reimplement git-flow — it delegates to the
git-flow CLI you have installed.

## Contributing

Issues and pull requests are welcome at
<https://github.com/joaoduarte19/GitExtensions.GitFlow>.

When the targeted Git Extensions version changes, please follow [semantic
versioning](https://semver.org/) and keep `GitExtensions.Extensibility` pinned to a single Git
Extensions major version — the plugin API can change between releases.

## License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.

## Acknowledgements

- Based on the original **GitFlow** plugin from the
  [Git Extensions](https://github.com/gitextensions/gitextensions) project, which was removed in Git
  Extensions 7.0 by [gitextensions/gitextensions#12894](https://github.com/gitextensions/gitextensions/pull/12894).
- The git-flow branching model by [Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/)
  ([nvie/gitflow](https://github.com/nvie/gitflow)).
- Scaffolded from the
  [Git Extensions Plugin Template](https://github.com/gitextensions/gitextensions.plugintemplate).
- Some icons by [Yusuke Kamiyamane](http://p.yusukekamiyamane.com).
