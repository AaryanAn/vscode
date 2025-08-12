## VS Code (Code - OSS) Selfhost Setup on macOS

This is a concise reference for setting up, building, running, and contributing to this VS Code fork on macOS.

### Prerequisites
- **Git**
- **Node.js via nvm**: Use the version in `.nvmrc` (currently `22.17.0`).
- **npm**: Comes with Node.
- **Python 3**: For `node-gyp`.
- **Xcode Command Line Tools (CLT)**: Provides `clang`, `make`, etc.

Install/align tools:
```bash
# Install/ensure nvm is installed (see https://github.com/nvm-sh/nvm)
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

# Use the repo's Node version
nvm install
nvm use

# Install Xcode Command Line Tools (GUI will appear if not present)
xcode-select --install
```

If CLT is broken or missing receipts (node-gyp fails complaining about Xcode/CLT), repair it:
```bash
sudo rm -rf /Library/Developer/CommandLineTools
sudo xcode-select --reset
softwareupdate --list | grep -A1 "Command Line Tools"  # find latest label
sudo softwareupdate -i "Command Line Tools for Xcode-<VERSION>" --agree-to-license
```
Verify:
```bash
xcodebuild -version || true   # will warn that full Xcode isn’t selected (OK)
pkgutil --pkg-info=com.apple.pkg.CLTools_Executables
```

Optional: auto-load nvm in new shells (zsh):
```bash
cat >> ~/.zshrc <<'EOF'
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion"
EOF
```

### Repository remotes
```bash
# origin: your fork (already set when cloning)
git remote -v

# add upstream once
git remote add upstream https://github.com/microsoft/vscode.git
```

### Install dependencies
From the repo root (`vscode` folder):
```bash
nvm use
rm -rf node_modules ~/Library/Caches/node-gyp
npm cache clean --force
npm ci
```

### Build
- One-time compile:
```bash
npm run compile
```
- Development watch (preferred):
```bash
npm run watch
```
- Web bits (only if working on web UI/built-in web extensions):
```bash
npm run watch-web
```

### Run
```bash
./scripts/code.sh            # launch Code - OSS
./scripts/code-cli.sh --version  # CLI sanity check
```

Tip: After code changes, use “Reload Window” from the Command Palette (map to Cmd+R) to refresh the dev window.

### Tests and lint (optional)
```bash
./scripts/test.sh
npm run eslint
```

### Syncing with upstream and contributing
```bash
# Update local main
git checkout main
git fetch upstream
git merge upstream/main

# Create a feature branch
git checkout -b feat/<your-feature>
# ... make edits ...
git push -u origin HEAD

# Open a PR against microsoft/vscode and link the issue
```

### Troubleshooting
- Build errors after updates:
```bash
git clean -xfd   # WARNING: deletes untracked files
npm ci
```
- node-gyp/Xcode errors on macOS: repair CLT as shown above.
- Electron app not valid: ensure you ran `npm run watch` or `npm run compile` before `./scripts/code.sh`.

### Notes
- Clone path should avoid spaces.
- Keep Node aligned with `.nvmrc`.
- On first `npm ci`, native modules may compile; this can take a bit.


