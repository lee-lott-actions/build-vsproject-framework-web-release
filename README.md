# Build Visual Studio .NET Framework Web Project Release Action

This GitHub Action builds a Visual Studio .NET Framework web project, publishes it to a specified directory, packages the output into a zip file, and creates a GitHub release with semantic versioning and release notes. It fully automates the build and release process for classic .NET Framework web applications, including support for pre-releases.

---

## Features
- Builds a Visual Studio .NET Framework web project (`.csproj` or `.vbproj`) using MSBuild.
- Publishes the project to a `_PublishedWebsites` directory and packages the output into a zip file.
- Restores NuGet packages using the provided solution file before the build.
- Creates a GitHub release with automated release notes and attaches build artifacts.
- Supports pre-release versioning with a custom suffix.
- Adds a job summary with release details in the workflow run.
- Includes the release date and time in Central Daylight Time (CDT).

---

## Inputs

| Name                | Description                                                                                                                             | Required |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------|----------|
| `project-file-path` | The path to the Visual Studio .NET Framework project file (`.csproj` or `.vbproj`) to be built.                                         | Yes      |
| `project-name`      | The name of the Visual Studio project (usually the .csproj/.vbproj file name without its extension). Used for finding publish output.   | Yes      |
| `solution-path`     | The relative path to the solution file (`.sln`) for restoring NuGet packages, e.g., `src/MySolution.sln`.                              | Yes      |
| `prerelease`        | Mark the release as a pre-release (`true`/`false`).                                                                                     | Yes      |
| `prerelease-suffix` | The suffix to append to the version number for pre-releases. For non-pre-releases, the version is based on the latest pre-release with this suffix. | Yes      |
| `package-name`      | The base name for the zip file containing the build artifacts.                                                                          | Yes      |
| `configuration`     | The build configuration for the .NET project (e.g., Debug, Release).                                                                   | Yes      |
| `token`             | The GitHub token used for authentication to create the release.                                                                         | Yes      |

---

## Outputs

| Name            | Description                                                                                 |
|-----------------|---------------------------------------------------------------------------------------------|
| `version-tag`   | The new tag created for the release (e.g., `v1.0.0`).                                      |
| `version-number`| The new tag with the "v" prefix removed (e.g., `1.0.0`).                                   |
| `zip-file-name` | The name of the zip file containing the build artifacts, e.g., `my-package.1.0.0.zip`.     |
| `zip-file-path` | The path to the zip file containing the build objects.                                      |
| `build-status`  | The build/release outcome: Release Created, No Release, or Release Failed.                 |

---

## Usage

**Important**:  
- Run this action on a `windows-latest` runner for MSBuild and .NET Framework support.
- Designed to run on PR merges or pushes to your default branch.
- Ensure the runner environment includes required MSBuild tooling and Visual Studio (as provided by GitHub's hosted Windows runners).

### Example Workflow

```yaml
name: Build and Release .NET Framework Web Project

on:
  pull_request:
    types: [closed]
  push:
    branches: [ main ]

jobs:
  build-and-release:
    # Only run for merged PRs, you can adjust this as necessary
    if: github.event.pull_request.merged == true || github.event_name == 'push'
    runs-on: windows-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Build Web Project and Create Release
        uses: lee-lott-actions/build-vsproject-framework-web-release@v1
        with:
          project-file-path: 'src/MyWebApp/MyWebApp.csproj'
          project-name: 'MyWebApp'
          solution-path: 'src/MyWebApp.sln'
          prerelease: 'false'
          prerelease-suffix: 'beta'
          package-name: 'my-web-app'
          configuration: 'Release'
          token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Notes
- The action uses the `_PublishedWebsites` folder as output for published web files; make sure this matches your project’s publish behavior.
- For pre-releases, a git tag is created but a GitHub Release is not published.
- For normal releases, both a tag and a Release (with attached zip) are created.
- Release notes and job summary output are included automatically.
- The action can be triggered on PR merges or direct pushes, giving you flexibility in your release flow.

---
