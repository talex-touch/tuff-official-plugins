# Tuff Plugin Repository Integration Guide

This document outlines the process for a client application (e.g., the Tuff app) to discover, install, and manage plugins from this repository.

## Overview

The integration process relies on a single manifest file, `plugins.json`, located at the root of the repository. This file acts as the single source of truth for all available plugins and their versions.

The typical workflow is as follows:
1.  **Fetch Manifest**: The client periodically fetches `plugins.json`.
2.  **Display Plugins**: The client parses the manifest to display a list of available plugins to the user (e.g., in a plugin marketplace).
3.  **Install Plugin**: When a user chooses to install a plugin, the client downloads the correct version of the `.tpex` file using the path provided in the manifest.
4.  **Check for Updates**: The client compares its locally installed plugin versions against the latest versions listed in the manifest to detect available updates.

---

## Step-by-Step Integration

### 1. Fetching the Plugin Manifest

The client must first retrieve the central manifest file.

-   **Endpoint**: The client should be configured to fetch `plugins.json` from the root of this repository. The raw file URL would be something like:
    `https://github.com/YourUsername/tuff-official-plugins/raw/main/plugins.json`
-   **Caching**: To reduce network traffic and improve performance, the client should implement a caching mechanism. It is highly recommended to use HTTP headers like `ETag` or `Last-Modified` to only download the file when it has actually changed.
-   **Frequency**: The client can fetch this file on startup and then periodically (e.g., once every 24 hours).

### 2. Parsing and Displaying Plugins

Once `plugins.json` is fetched and parsed, the client has an array of plugin objects. For each plugin, the client can use the following fields to populate its UI:

-   `name`: The display name of the plugin.
-   `description`: A short summary of what the plugin does.
-   `author`: The plugin's author.
-   `category`: Used to group plugins in the UI (e.g., "Tools", "Themes").
-   `version`: The latest available version string.

### 3. Installing a Plugin

When a user initiates an installation for a plugin with a specific `id`:

1.  **Find the Plugin**: The client locates the corresponding object in the `plugins.json` array by its `id`.
2.  **Construct URL**: It reads the `path` property (e.g., `plugins/tools/com.tuffex.translation/1.0.0-Alpha/touch-translation-1.0.0-Alpha.tpex`).
3.  **Download**: The client prepends the base repository URL to this relative path to get the full download URL for the `.tpex` file.
4.  **Install**: After downloading, the client installs the `.tpex` package into its local plugins directory.
5.  **Store Metadata**: The client **must** store the `id` and the installed `version` locally for future update checks.

### 4. Updating an Existing Plugin

To check for updates:

1.  **Fetch Latest Manifest**: The client gets the latest `plugins.json`.
2.  **Compare Versions**: For each locally installed plugin, the client finds its entry in the manifest using the `id`. It then compares the locally stored version string with the `version` string from the manifest.
    *(It is recommended to use a semantic versioning (semver) comparison library for robust checking).*
3.  **Prompt for Update**: If the manifest version is newer than the local version, the client should notify the user that an update is available.
4.  **Perform Update**: The update process is identical to the installation process: download the new version's `.tpex` file and replace the old one.

### 5. (Optional) Version Rollback

The `versions` array within each plugin object provides a history of available versions. This allows for advanced features like version rollback.

-   If a user encounters a bug with a new update, the client could present a list of older versions from the `versions` array.
-   The user can then select a specific version to downgrade to, and the client would use the `path` from that version's object to download and install it.

This structured approach ensures a robust and maintainable plugin ecosystem.