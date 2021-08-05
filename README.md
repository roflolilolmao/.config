# .config

This repository contains a minimal configuration to use neovim and zsh on WSL.

## Install

### Fonts

Nerd fonts are fonts that contain thousands of useful glyphs. Using one will
make everything prettier.

I'm using FiraCode because it comes with ligatures.

[Nerd fonts](https://www.nerdfonts.com/font-downloads).

### WSL

Run the following commands.

```zsh
sudo apt update
sudo apt upgrade
sudo apt install zsh unzip
  
chsh -s /bin/zsh [user]

# Set up your SSH stuff if necessary
# https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh

git clone git@github.com:[github_username]/.config --recursive
cd .config

# Zsh only looks for `~/.zshenv`
ln -s $(pwd)/zsh/.zshenv ~/.zshenv

curl -L -o nvim.appimage https://github.com/neovim/neovim/releases/download/v0.5.0/nvim.appimage
chmod u+x nvim.appimage
mkdir -p ~/.local/bin/
mv nvim.appimage ~/.local/bin/nvim

# Clipboard provider for WSL, used by nvim
# https://github.com/neovim/neovim/wiki/FAQ#how-to-use-the-windows-clipboard-from-wsl
curl -sLo/tmp/win32yank.zip https://github.com/equalsraf/win32yank/releases/download/v0.0.4/win32yank-x64.zip
unzip -p /tmp/win32yank.zip win32yank.exe > /tmp/win32yank.exe
chmod +x /tmp/win32yank.exe
mv /tmp/win32yank.exe ~/.local/bin/win32yank.exe
```

### Changing the repository's location

This repository's default folder is `$HOME/.config`. If you want to put it
elsewhere, you must edit `zsh/.zshenv` to point to the correct folder. Change
this line:

```zsh
export XDG_CONFIG_HOME="$HOME/.config"
```

Now restart the shell and everything should be set.

## Configuration

### SSH

To start up the ssh agent with the shell, you can add these two lines to
`zsh/.zshrc`:

```zsh
eval $(ssh-agent -s) > /dev/null
ssh-add ~/.ssh/your_private_key 2> /dev/null
```

### Zsh

- Customize the prompt by running `p10k configure`.
- Define aliases in `zsh/aliases.zsh`.

### wsl.conf

Optionally, [the Windows %PATH% can be removed from the linux
$PATH](https://stackoverflow.com/a/63195953/12474293). Some programs will slow
down massively when they try to parse the folders in `$PATH`. The downside of
doing this is that Windows programs (such as `explorer.exe`) can't be used
inside WSL. `/etc/wsl.conf` is the file containing the configuration of WSL
itself, this is where this option can be set. This repository contains a file
we can symlink:

```zsh
cd ~/.config
sudo ln -sf $(pwd)/wsl.conf /etc/wsl.conf
```

## Misc

### Update Submodules

```zsh
git submodule update --remote
```

### Terminals Input

Terminals handle input in a way that's not intuitive. Programs in the Windows
Terminal will not receive some inputs because of how terminals handle them. For
example, typing `<C-CR>` would send `<CR>` to the program running in the
terminal. The following escape sequences can be added in the terminal settings
to send the input regardless of the terminal's behavior:

```json
"actions":
[
  {
      "command":
      {
          "action": "sendInput",
          "input": "\u001b[27;2;13~"
      },
      "keys": "shift+enter",
      "name": ""
  },
  {
      "command":
      {
          "action": "sendInput",
          "input": "\u001b[27;5;13~"
      },
      "keys": "ctrl+enter",
      "name": ""
  }
]
```

Generally:

| output | input |
| -- | -- |
| \<C-x> | \u001b[27;5;X~ |
| \<S-x> | \u001b[27;2;X~ |
| \<C-S-x> | \u001b[??????~ |

Where `x` is the wanted char and `X` its ascii code.

Some resources regarding this:

- [ascii table](https://www.asciitable.com/)
- terminal
  - [escape sequences](https://github.com/microsoft/terminal/pull/8330)
  - [some illegible stuff](https://github.com/microsoft/terminal/issues/8931)
- escape sequences
  - [shift tab](https://unix.stackexchange.com/questions/238406/why-does-shift-tab-result-in-escape-in-the-terminal)
  - [ctrl-arrow](https://stackoverflow.com/questions/7767702/what-is-terminal-escape-sequence-for-ctrl-arrow-left-right-in-term-linu)
- terminfo
  - [wiki](https://en.wikipedia.org/wiki/Terminfo)
  - [man-terminfo](https://www.man7.org/linux/man-pages/man5/terminfo.5.html)
