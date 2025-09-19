# ZSH

1. **Install the tools (macOS / Homebrew):**

```bash
brew install zsh git zoxide fzf eza bat ripgrep fd git-delta
# optional but nice
brew install jq tree
# set up fzf keybindings & completion
"$(brew --prefix)/opt/fzf/install" --key-bindings --completion --no-bash --no-fish --no-update-rc
```

2. **Install a Nerd Font** (for icons & Powerlevel10k):

* In iTerm2: Preferences → Profiles → Text → Font → pick a Nerd Font (e.g., **MesloLGS NF**).
  (Powerlevel10k will also offer to install MesloLGS NF during its wizard.)

3. **Add an ultra-fast plugin manager (Antidote):**

```bash
git clone --depth=1 https://github.com/mattmc3/antidote.git ${ZDOTDIR:-$HOME}/.antidote
```

4. **Create your plugin list at `~/.zsh_plugins.txt`:**

```text
romkatv/powerlevel10k
ohmyzsh/ohmyzsh path:plugins/git
zsh-users/zsh-autosuggestions
zdharma-continuum/fast-syntax-highlighting
zsh-users/zsh-completions
Aloxaf/fzf-tab
```

5. **Drop-in `~/.zshrc` (copy/paste this whole block):**

```zsh
# 0) Homebrew path (Apple Silicon + Intel)
if [[ -d /opt/homebrew ]]; then
  eval "$(/opt/homebrew/bin/brew shellenv)"
elif [[ -d /usr/local/Homebrew ]]; then
  eval "$(/usr/local/bin/brew shellenv)"
fi

# 1) Powerlevel10k instant prompt (keep near top)
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# 2) History & shell behavior
HISTFILE=$HOME/.zsh_history
HISTSIZE=200000
SAVEHIST=200000
setopt share_history inc_append_history hist_ignore_all_dups
setopt autopushd pushdignoredups pushdminus
setopt auto_cd extendedglob interactivecomments
unsetopt beep

# 3) Aliases & quality-of-life
alias ls='eza --icons --group-directories-first --git'
alias ll='ls -lah'
alias la='ls -a'
alias cat='bat -p'
alias grep='rg --hidden --smart-case'
alias please='sudo $(fc -ln -1)'
alias ..='cd ..'
alias ...='cd ../..'

# Git sugar
alias gst='git status -sb'
alias ga='git add'
alias gc='git commit -v'
alias gcm='git commit -m'
alias gco='git checkout'
alias gcb='git checkout -b'
alias gp='git push'
alias gl='git pull'
alias gd='git diff'
alias gds='git diff --staged'
alias glg='git log --oneline --graph --decorate --all'

# 4) fzf defaults (fast, pretty previews)
export FZF_DEFAULT_COMMAND='fd --hidden --follow --exclude .git'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_DEFAULT_OPTS='--height=40% --layout=reverse --border --preview-window=right:60%:wrap'
export FZF_TMUX=1

# fzf keybindings & completion
if command -v fzf >/dev/null 2>&1; then
  [[ -r "$(brew --prefix)/opt/fzf/shell/key-bindings.zsh" ]] && source "$(brew --prefix)/opt/fzf/shell/key-bindings.zsh"
  [[ -r "$(brew --prefix)/opt/fzf/shell/completion.zsh"    ]] && source "$(brew --prefix)/opt/fzf/shell/completion.zsh"
fi

# 5) zoxide (blazing-dir-jumps). Use 'z foo' or just 'z'
eval "$(zoxide init zsh --cmd z)"

# 6) Antidote plugin manager (fast)
source "${ZDOTDIR:-$HOME}/.antidote/antidote.zsh"
# (Re)build plugin bundle if missing or list changed
if [[ ! -f ${ZDOTDIR:-$HOME}/.zsh_plugins.zsh || ~/.zsh_plugins.txt -nt ~/.zsh_plugins.zsh ]]; then
  antidote bundle < ~/.zsh_plugins.txt > ~/.zsh_plugins.zsh
fi
source ~/.zsh_plugins.zsh

# 7) Completion (init AFTER plugins so extra completions are in $fpath)
autoload -Uz compinit
zmodload zsh/complist
# cache compdump for speed
ZSH_COMPDUMP="${XDG_CACHE_HOME:-$HOME/.cache}/zsh/zcompdump-$ZSH_VERSION"
mkdir -p "${ZSH_COMPDUMP:h}"
compinit -d "$ZSH_COMPDUMP"

# 8) fzf-tab settings (fzf-powered completion UI)
zstyle ':fzf-tab:*' switch-group '<' '>'
zstyle ':fzf-tab:complete:file:*' fzf-preview \
  '([[ -d $realpath ]] && eza -1 --color=always --group-directories-first --icons $realpath) || (bat --style=numbers --color=always --line-range :500 --paging=never $realpath 2>/dev/null || file -r -- $realpath)'

# nice selection menu for classic completion
zstyle ':completion:*' menu select
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Za-z}' 'r:|[._-]=** r:|=**' 'l:|=* r:|=*'
zstyle ':completion:*' list-colors ${(s.:.)LS_COLORS}

# 9) Keymap: Emacs-style (or use: bindkey -v for vi)
bindkey -e

# 10) Powerlevel10k prompt config (run: p10k configure)
[[ -f ~/.p10k.zsh ]] && source ~/.p10k.zsh

# 11) asdf-vm setup
export PATH="${ASDF_DATA_DIR:-$HOME/.asdf}/shims:$PATH"
fpath=(${ASDF_DATA_DIR:-$HOME/.asdf}/completions $fpath)
autoload -Uz compinit && compinit

# 12) direnv setup
eval "$(direnv hook zsh)" 
```

6. **Setup asdf completions:**
```bash
mkdir -p "${ASDF_DATA_DIR:-$HOME/.asdf}/completions"
asdf completion zsh > "${ASDF_DATA_DIR:-$HOME/.asdf}/completions/_asdf"

```

7. **Build the plugin bundle once and start ZSH:**

```bash
antidote bundle < ~/.zsh_plugins.txt > ~/.zsh_plugins.zsh
exec zsh
```

8. **Run the prompt wizard** (this is where the beauty happens):

```bash
p10k configure
```

Pick “lean” or “rainbow”, enable icons, and latency-friendly options. You can rerun anytime.

9. **Configure asdf-vm**
export PATH="${ASDF_DATA_DIR:-$HOME/.asdf}/shims:$PATH"

10. **(Optional) Prettier Git diffs with delta** – add to `~/.gitconfig`:

```ini
[core]
  pager = delta
[interactive]
  diffFilter = delta --color-only
[delta]
  navigate = true
  line-numbers = true
  side-by-side = true
[merge]
  conflictstyle = zdiff3
[diff]
  colorMoved = default
```

---

## Deeper Dive (what you got & why it’s fast)

* **Antidote** loads plugins from a compiled bundle for near-instant startup. Edit `~/.zsh_plugins.txt`, then rebuild:
  `antidote bundle < ~/.zsh_plugins.txt > ~/.zsh_plugins.zsh`.

* **Powerlevel10k**: best-in-class prompt performance + stunning visuals. It shows Git status instantly and can display node/python/dir info without lag.

* **Great autocomplete**:

  * **zsh-completions** adds tons of extra completion specs (brew, git, docker, etc.).
  * **fzf-tab** replaces the default completion menu with an fzf-powered picker, complete with file previews via **bat**/**eza**.
  * **zsh-autosuggestions** gives ghost text suggestions from history as you type—accept with → or `End`.

* **Speed tricks baked in**:

  * Completion cache (`compinit -d`) and static plugin bundle.
  * Minimal plugins, but high impact.
  * `fd`, `ripgrep`, and `fzf` configured as your default fuzzy engine.

* **zoxide**: jump to any directory you’ve visited with `z <name>` (e.g., `z proj`, `z dotfiles`). It learns as you move.

* **Git UX**:

  * `ohmyzsh git` plugin gives handy aliases + completion.
  * Aliases in `.zshrc` for your muscle memory (`gst`, `gco`, `glg`, etc.).
  * `delta` makes diffs readable and fast.

* **Quality-of-life**:

  * **eza** (modern `ls`) with icons and Git indicators.
  * **bat** (modern `cat`) with syntax highlighting and line numbers in previews.
  * Thoughtful `setopt` and history settings so your past commands are always at hand without duplicates.

---

If you want, I can also tailor this for **vi keybindings**, add **conda/pyenv/nvm** integrations, or swap **fzf-tab** for **zsh-autocomplete**’s inline suggestions (heavier but very slick). But the setup above already hits your goals: **beautiful**, **fast**, **Git-savvy**, **killer autocomplete**, and **zoxide** baked in.
