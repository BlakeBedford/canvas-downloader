# Canvas Downloader

A command-line tool to download and organize all your Canvas course materials—files, syllabi, pages, modules, assignments, discussions, and announcements—into a clean local folder structure. Made in async Rust⚡.

This repository is a maintained fork of [Blake Bedford's canvas-downloader fork](https://github.com/BlakeBedford/canvas-downloader), which itself is based on the original [bnjmnt4n/canvas-downloader](https://github.com/bnjmnt4n/canvas-downloader).

## Build From Source

This fork is intended to be built from source.

```bash
git clone https://github.com/BlakeBedford/canvas-downloader.git
cd canvas-downloader
cargo build --release
```

## Quick Start

### 1. Create Configuration File

You can copy the [example config](examples/config.toml) into one of the **config file locations (searched in order):**

1. Custom path via `--config` option
1. `canvas-downloader.toml` in current directory
1. `config.toml` in platform-specific config directory:
   - Linux: `~/.config/canvas-downloader/config.toml`
   - macOS: `~/.config/canvas-downloader/config.toml` or `~/Library/Application Support/canvas-downloader/config.toml`
   - Windows: `%APPDATA%\canvas-downloader\config.toml`

Then modify it to your Canvas instance URL and access token.

#### How to get your token

- Log in to Canvas → Account → Settings → **New Access Token**

### 2. Discover Your Courses

Run the tool to see which courses are available:

```shell
$ canvas-downloader
Please provide either Term ID(s) via -t or course name(s)/code(s) via -c
Term ID    | Course Code | Course Name
-----------------------------------------------------------
115        | CS1101S     | Programming Methodology
           | CS1231S     | Discrete Structures
-----------------------------------------------------------
120        | CS2040S     | Data Structures and Algorithms
           | CS2030      | Programming Methodology II
-----------------------------------------------------------
125        | CS3230      | Design and Analysis of Algorithms
```

### 3. Download Your Courses

You can download courses by term ID or by course name/code:

**Download by terms (all courses in specific terms):**

```shell
$ canvas-downloader -t 115 120
```

**Download by course names and/or codes (specific courses only):**

```shell
$ canvas-downloader -c CS1101S "Introduction to Data Structures"
```

**Combine both (courses matching both criteria):**

```shell
$ canvas-downloader -t 115 -c CS1101S
```

The tool will show you all files to be downloaded with their sizes, then ask for confirmation before proceeding. Downloads are organized by course, preserving Canvas's folder structure.

> **Note:** Course name matching is exact match - use the exact course code (e.g., "CS1101S") or the exact course name as shown in the discovery step.

## Recent Fixes

- Canvas module `Quiz` items are now handled as local `.url` shortcuts instead of being logged as unsupported.
- Canvas pagination no longer duplicates query parameters across requests or panics when a `current` link relation is missing.
- Shared Canvas API requests now retry transient failures and use a longer per-attempt timeout for slower endpoints.
- Configured Canvas base URLs are normalized before requests are built, preventing malformed `//api/...` paths.
- Panopto downloads now fail gracefully when a playlist has no variants or no segments instead of aborting the run.

## What Gets Downloaded

- [x] Files
- [x] Modules
- [x] Syllabi (in HTML and JSON)
- [x] Assignments (in HTML and JSON)
- [x] Discussions and announcements (in HTML and JSON)
- [x] Pages (in HTML and JSON)
- [x] User information (in JSON)
- [ ] Panopto lecture videos (seems still buggy)

## Common Workflows

### Filter What You Download

Create a `.canvasignore` file in your current directory to skip certain files using `.gitignore` syntax:

```shell
# Ignore all videos
*.mp4
*.mov

# Ignore specific courses
/CS1101S/

# Ignore lecture recordings folder
lecture-recordings/
```

The tool automatically loads `.canvasignore` from the current directory if it exists. You can also specify a custom ignore file with `-i`:

```shell
$ canvas-downloader -t 115 -i custom-ignore.txt
```

See the [example file](examples/.canvasignore) for more patterns.

### Keep Your Files Updated

Use `-n` to overwrite local files with newer versions from Canvas:

```shell
$ canvas-downloader -t 115 -n
```

By default, existing local files won't be overwritten even if Canvas has newer versions.

### Choose Download Location

Specify a custom folder with `-d`:

```shell
$ canvas-downloader -t 115 -d ~/Canvas
```

### See Debug Information

Use `-v` to enable verbose output for troubleshooting:

```shell
# Enable debug logging
$ canvas-downloader -t 115 -v
```

Without `-v`, only important progress messages are shown (info level).

## All Options

```
Usage: canvas-downloader [OPTIONS]

Options:
      --config <FILE>                Path to config file (default: platform-specific config locations)
  -d, --destination-folder <FOLDER>  Download location [default: .]
  -n, --download-newer               Overwrite local files with newer Canvas versions
  -t, --term-ids <ID>...             Term IDs to download
  -c, --course-names <NAME>...       Course names or codes to download - exact match
  -i, --ignore-file <FILE>           Path to ignore patterns file [default: .canvasignore]
      --dry-run                      Preview downloads without executing
      --no-raw                       Do not save raw JSON responses
      --no-submissions               Do not download assignment submission files
  -v, --verbose                      Enable debug logging
  -h, --help                         Print help
  -V, --version                      Print version
```
