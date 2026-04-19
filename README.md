# nixos-xmonad

My personal NixOS and XMonad configuration.

## Structure

- **nixos/** — system configuration used for `/etc/nixos`
- **xmonad/** — XMonad configuration used for `~/.xmonad`

## Usage

Clone the repo:

```bash
git clone git@github.com:Vittalbigamudra/nixos-xmonad.git ~/dotfiles
```

Link the configs:

```bash
sudo ln -sf ~/dotfiles/nixos /etc/nixos
ln -sf ~/dotfiles/xmonad ~/.xmonad
```

Rebuild NixOS:

```bash
sudo nixos-rebuild switch
```

Restart XMonad:

```bash
xmonad --recompile && xmonad --restart
```

## Goals

- Fully reproducible system
- Declarative window manager setup
- Clean, minimal configuration
