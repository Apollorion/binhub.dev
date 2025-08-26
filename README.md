# BinHub - Universal Binary Distribution

BinHub is a centralized platform for distributing pre-compiled binaries across multiple architectures. It provides a simple way to host and distribute popular command-line tools and utilities for automated deployments, CI/CD pipelines, and development environments.

## 🏗️ Architecture

- **YAML Metadata Files**: Define binary sources, architectures, and metadata
- **Python Processor**: Downloads, extracts, and processes binaries
- **Nested Directory Structure**: Organizes binaries by letter/name/version/os-architecture
- **Hierarchical APIs**: Multi-level JSON APIs for discovery and navigation
- **Cloud Storage**: Uploads to Cloudflare R2 or S3 for global distribution

## 📁 Repository Structure

```
binaries/
├── 0-9/           # Binaries starting with numbers
├── a/             # Binaries starting with 'a'
├── b/             # Binaries starting with 'b'
...
├── z/             # Binaries starting with 'z'
│   └── .gitkeep   # Keep empty directories in git
processor.py       # Main processing script
requirements.txt   # Python dependencies
README.md         # This file
```

## 📝 YAML Schema

Each binary is defined by a YAML file in the appropriate alphabetical directory:

```yaml
name: example-binary
description: Short description of the binary
homepage: https://example.com
repository: https://github.com/user/repo
license: MIT
version: "1.0.0"

architectures:
  linux-amd64:
    url: https://github.com/user/repo/releases/download/v1.0.0/binary-linux-amd64.tar.gz
    type: tar.gz                    # Archive type: zip, tar.gz, tgz, tar, or raw
    binary_path_in_archive: bin/binary  # Path to binary inside archive (not needed for raw)
    sha256: a1b2c3d4e5f6...         # SHA256 checksum (optional but recommended)
  
  darwin-amd64:
    url: https://github.com/user/repo/releases/download/v1.0.0/binary-darwin-amd64
    type: raw                       # Raw binary, no extraction needed
    sha256: b2c3d4e5f6a1...

tags:
  - cli
  - development
  - tools
```

### Supported Archive Types

- `raw`: Direct binary download (no extraction)
- `zip`: ZIP archive
- `tar.gz` / `tgz`: Gzipped tar archive
- `tar`: Plain tar archive

### Architecture Naming

Use these standard OS-architecture names in YAML files:
- `linux-amd64`: Linux 64-bit Intel/AMD
- `linux-arm64`: Linux 64-bit ARM
- `darwin-amd64`: macOS Intel
- `darwin-arm64`: macOS Apple Silicon
- `windows-amd64`: Windows 64-bit

**Important**: Keep the full `{os}-{arch}` format as binaries are OS-specific even with the same instruction set. A Linux binary cannot run on macOS and vice versa due to different binary formats (ELF vs Mach-O), system calls, and library dependencies.

## 🚀 Usage

### Running the Processor

1. Install dependencies:
```bash
pip install -r requirements.txt
```

2. Run the processor:
```bash
python processor.py
```

The processor will:
- Scan all YAML files in the `binaries/` directory
- Download and extract binaries as needed
- Create nested directory structure: `{letter}/{name}/{version}/{os-arch}/`
- Generate hierarchical API files at each level
- Generate static HTML index site

### Nested Output Structure

```
output/
├── api.json                    # Root API - lists available letters
├── index.html                  # Static HTML site
├── j/                          # First letter directory
│   ├── api.json               # Letter API - lists binaries starting with 'j'
│   └── jq/                    # Binary name directory
│       ├── api.json           # Binary API - shows metadata and versions
│       └── 1.6/               # Version directory
│           ├── api.json       # Version API - shows architectures
│           ├── linux-amd64/   # OS-Architecture directory
│           │   └── jq         # Binary file (Linux ELF format)
│           ├── darwin-amd64/  # OS-Architecture directory  
│           │   └── jq         # Binary file (macOS Mach-O format)
│           └── windows-amd64/
│               └── jq.exe     # Binary file (Windows PE format)
└── g/
    ├── api.json
    └── gh/
        ├── api.json
        └── 2.40.1/
            ├── api.json
            ├── linux-amd64/
            │   └── gh
            ├── darwin-amd64/
            │   └── gh
            └── windows-amd64/
                └── gh.exe
```

## 📡 Hierarchical API Structure

The processor generates multiple API levels for easy discovery:

### Root API (`/api.json`)
Lists all available first letters/numbers:
```json
{
  "version": "1.0",
  "directories": ["j", "g", "k"]
}
```

### Letter API (`/j/api.json`)
Lists all binaries starting with that letter:
```json
{
  "version": "1.0",
  "binaries": ["jq", "jwt"]
}
```

### Binary API (`/j/jq/api.json`)
Shows binary metadata and available versions:
```json
{
  "name": "jq",
  "description": "Command-line JSON processor",
  "homepage": "https://stedolan.github.io/jq/",
  "repository": "https://github.com/stedolan/jq",
  "license": "MIT",
  "tags": ["cli", "json", "parsing"],
  "versions": ["1.6", "1.7"]
}
```

### Version API (`/j/jq/1.6/api.json`)
Shows architectures and direct download URLs:
```json
{
  "name": "jq",
  "version": "1.6",
  "architectures": {
    "linux-amd64": {
      "url": "/j/jq/1.6/linux-amd64/jq",
      "size": 3953824,
      "sha256": "af986793a515d500ab2d35f8d2aecd656e764504b789b66d7e1a0b727a124c44"
    },
    "darwin-amd64": {
      "url": "/j/jq/1.6/darwin-amd64/jq",
      "size": 864040,
      "sha256": "5c0a0a3ea600f302ee458b30317425dd9632d1ad8882259fcaf4e9b868b2b1ef"
    },
    "windows-amd64": {
      "url": "/j/jq/1.6/windows-amd64/jq.exe",
      "size": 1234567,
      "sha256": "9a8b7c6d5e4f3g2h1i0j9k8l7m6n5o4p3q2r1s0t9u8v7w6x5y4z3a2b1c0d9e8f"
    }
  }
}
```

### Direct Binary Access
Binaries are accessible at predictable URLs with full OS-architecture specification:
- `/j/jq/1.6/linux-amd64/jq` (Linux binary)
- `/j/jq/1.6/darwin-amd64/jq` (macOS binary)
- `/g/gh/2.40.1/windows-amd64/gh.exe` (Windows binary)
- `/k/kubectl/1.28.0/linux-arm64/kubectl` (Linux ARM64 binary)

## 🤝 Contributing

1. Fork this repository
2. Add a YAML file in the appropriate directory (e.g., `binaries/g/gh.yaml` for GitHub CLI)
3. Follow the YAML schema with proper URLs and checksums
4. Test locally with `python processor.py`
5. Submit a pull request

### Adding a New Binary

1. Determine the first character of the binary name
2. Create `binaries/{character}/{binary-name}.yaml`
3. Fill out all required fields
4. Include SHA256 checksums when possible
5. Test that all download URLs work

### Guidelines

- Use official release URLs when possible
- Always include SHA256 checksums for security
- Keep descriptions concise but informative
- Add relevant tags for discoverability
- Follow the existing naming conventions

## 🔧 Development

### Local Testing

```bash
# Install dependencies
pip install -r requirements.txt

# Add test binaries (small ones for testing)
# Run processor
python processor.py

# Check output structure
find output -name "*.json" | sort
ls -la output/j/jq/1.6/linux-amd64/
ls -la output/j/jq/1.6/darwin-amd64/
```

### Error Handling

The processor handles various error conditions:
- Invalid URLs or network failures
- Corrupted archives or missing files
- SHA256 checksum mismatches
- Malformed YAML files

Failed binaries are logged but don't stop processing of other binaries.

## 📜 License

MIT License - see LICENSE file for details.

## 🙏 Acknowledgments

Thanks to all the maintainers of the open source tools that this project helps distribute!