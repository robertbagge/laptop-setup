# ASDF

```bash
asdf plugin add nodejs
asdf plugin add bun
asdf plugin add python
asdf plugin add golang

cp asdf/.tool-versions ~
asdf install
```

## Golang

Add the following to .zshrc

```bash
. ${ASDF_DATA_DIR:-$HOME/.asdf}/plugins/golang/set-env.zsh
```
