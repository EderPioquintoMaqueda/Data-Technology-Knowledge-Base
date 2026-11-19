---
title: "Developer Environment - VS Code, Jupyter and Terminal"
tags: [tools, vscode, jupyter, terminal, setup]
---

# 💻 Developer Environment — VS Code, Jupyter and Terminal

## 🟢 Beginner: install the essentials (Windows)
1. **Python 3.12+** (python.org, tick "Add to PATH") — or `winget install Python.Python.3.12`.
2. **VS Code** — `winget install Microsoft.VisualStudioCode`.
3. **Git** — `winget install Git.Git` (includes Git Bash).
4. **WSL2 + Ubuntu** — `wsl --install` (a real Linux inside Windows).
5. **Docker Desktop** — uses WSL2.
6. **DBeaver Community** — universal database client.
7. **uv** — `pip install uv` (fast Python package manager).

### VS Code extensions
| Extension | Why |
|---|---|
| Python + Pylance | IntelliSense, type checking |
| Jupyter | run notebooks inside VS Code |
| Ruff | lint + format on save |
| GitLens | blame, history |
| Docker / Dev Containers | work inside containers |
| SQLTools (+ PostgreSQL driver) | run SQL from the editor |
| Rainbow CSV / Data Wrangler | inspect CSVs and DataFrames |
| Remote - SSH / WSL | edit code on servers / WSL |

### Must-know shortcuts
| Action | Shortcut |
|---|---|
| Command palette | `Ctrl+Shift+P` |
| Quick open file | `Ctrl+P` |
| Terminal | ``Ctrl+` `` |
| Multi-cursor | `Alt+Click` / `Ctrl+D` |
| Go to definition | `F12` |
| Rename symbol | `F2` |
| Comment line | `Ctrl+/` |
| Run cell (Jupyter) | `Shift+Enter` |

## 🟡 Intermediate: settings that matter
```jsonc
// settings.json
{
  "editor.formatOnSave": true,
  "[python]": { "editor.defaultFormatter": "charliermarsh.ruff" },
  "python.analysis.typeCheckingMode": "basic",
  "files.trimTrailingWhitespace": true,
  "jupyter.askForKernelRestart": false
}
```
### Notebook discipline
- Restart & Run All before sharing (no hidden state).
- Move reusable code into `src/` modules; notebooks only orchestrate & visualise.
- Strip outputs before committing (`nbstripout`).
- Use `%load_ext autoreload` + `%autoreload 2` to edit modules live.

### Debugging
Set breakpoints (F9), run with F5, inspect variables, use the Debug Console; in notebooks use "Debug Cell".

## 🔴 Advanced: reproducible environments
- **Dev Containers:** a `.devcontainer/devcontainer.json` defines the full toolchain (Python, PostgreSQL, extensions) — every teammate gets the same setup.
- **Remote development:** VS Code Remote-SSH to a GPU server; JupyterLab on a server through an SSH tunnel (`ssh -L 8888:localhost:8888 server`).
- **Dotfiles:** keep your `.bashrc`, `.gitconfig`, VS Code settings in a Git repo.
- **Terminal upgrades:** Windows Terminal, Oh My Posh/Starship prompt, `fzf`, `ripgrep (rg)`, `bat`, `htop`.

## 🗣️ Vocabulary | Vocabulario
| English | IPA | Sounds like (ES) | Español |
|---|---|---|---|
| editor | /ˈed.ɪ.t̬ɚ/ | **E**-di-ter | editor |
| extension | /ɪkˈsten.ʃən/ | ik-**STEN**-shon | extensión |
| shortcut | /ˈʃɔːrt.kʌt/ | **SHORT**-kat | atajo |
| breakpoint | /ˈbreɪk.pɔɪnt/ | **BRÉIK**-point | punto de interrupción |
| kernel | /ˈkɝː.nəl/ | **KER**-nel | núcleo (de Jupyter) |
| settings | /ˈset̬.ɪŋz/ | **SE**-tings | configuración |
| toolchain | /ˈtuːl.tʃeɪn/ | **TUL**-chein | cadena de herramientas |

🏠 [[00 - Tools Roadmap]]
