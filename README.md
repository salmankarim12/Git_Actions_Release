# Project Documentation

## Overview

This project uses Poetry for Python dependency management and GitHub Actions for automated versioning and releases.

## Getting Started

### Prerequisites

- Python 3.8+
- Poetry
- Git

### Installation

1. **Clone the Repository**:
    ```bash
    git clone https://github.com/your-username/your-repo.git
    cd your-repo
    ```

2. **Install Dependencies**:
    ```bash
    poetry install
    ```

3. **Activate the Environment**:
    ```bash
    poetry shell
    ```

## Contributing

### Commit Message Conventions

Use these prefixes in your commit messages to trigger the correct version increment:

- **`fix:`** for bug fixes (patch increment)
- **`feat:`** for new features (minor increment)
- **`breaking:`** for breaking changes (major increment)

**Examples**:
- `fix: correct typo in documentation`
- `feat: add new authentication feature`
- `breaking: remove deprecated endpoint`

## Versioning and Releases

We follow [Semantic Versioning](https://semver.org/), which uses the format `MAJOR.MINOR.PATCH`.

### Versioning Rules

- **Patch**: For bug fixes that do not break compatibility.
- **Minor**: For new features that are backwards-compatible.
- **Major**: For breaking changes that are not backwards-compatible.

## GitHub Actions Workflow

Our GitHub Actions workflow automates version updates and releases. It performs the following steps:

1. **Checkout Code**: Gets the latest code from the repository.
2. **Read Current Version**: Extracts the current version from `pyproject.toml`.
3. **Determine Increment Type**: Determines if the version should be major, minor, or patch based on the commit message.
4. **Increment Version**: Updates the version number accordingly.
5. **Update `pyproject.toml`**: Modifies the version in the `pyproject.toml` file.
6. **Commit Changes**: Commits the updated version and pushes the changes.
7. **Generate Tag Name**: Prepares the version tag for the release.
8. **Create Release**: Tags and creates a release on GitHub.

### Example Workflow File

Create a file named `.github/workflows/versioning-and-release.yml` with the following content:

```yaml
name: Update Version and Create Release

on:
  push:
    branches:
      - main

jobs:
  update_and_release:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Read current version
      id: read_version
      run: |
        VERSION=$(grep -oP '(?<=version = ")[^"]*' pyproject.toml)
        echo "VERSION=$VERSION" >> $GITHUB_ENV

    - name: Determine increment type
      id: determine_increment
      run: |
        COMMIT_MESSAGE=$(git log -1 --pretty=%B)
        if echo "$COMMIT_MESSAGE" | grep -q "^breaking:"; then
          echo "INCREMENT_TYPE=major" >> $GITHUB_ENV
        elif echo "$COMMIT_MESSAGE" | grep -q "^feat:"; then
          echo "INCREMENT_TYPE=minor" >> $GITHUB_ENV
        else
          echo "INCREMENT_TYPE=patch" >> $GITHUB_ENV
        fi

    - name: Increment version
      id: increment_version
      run: |
        IFS='.' read -r major minor patch <<< "${VERSION}"
        if [ "$INCREMENT_TYPE" = "major" ]; then
          new_version="$((major + 1)).0.0"
        elif [ "$INCREMENT_TYPE" = "minor" ]; then
          new_version="${major}.$((minor + 1)).0"
        else
          new_version="${major}.${minor}.$((patch + 1))"
        fi
        echo "NEW_VERSION=$new_version" >> $GITHUB_ENV

    - name: Update pyproject.toml
      run: |
        sed -i "s/version = \"${VERSION}\"/version = \"${NEW_VERSION}\"/" pyproject.toml

    - name: Commit changes
      run: |
        git config --global user.name 'GitHub Actions'
        git config --global user.email 'actions@github.com'
        git add pyproject.toml
        git commit -m "Bump version to ${NEW_VERSION}"
        git push origin HEAD:main
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    - name: Generate Tag Name
      id: generate_tag
      run: echo "RELEASE_TAG=${NEW_VERSION}" >> $GITHUB_ENV

    - name: Create Release
      id: create_release
      uses: actions/create-release@v1
      with:
        tag_name: ${{ env.RELEASE_TAG }}
        release_name: Release ${{ env.RELEASE_TAG }}
        draft: false
        prerelease: false
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
