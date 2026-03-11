# dotfiles

This uses stow to build symlinks to our ~ directory

## Install stow

yay -S stow

## Install packages

Use `stow -t ~ [package-name]` to install symlink for that package.

For more verbose output, use the `-vv` option - Yes, use two v's

## Additional packages

- **brightnessctl** - For handling brightness of keyboard backlighting. Might need to change device name in `hypridle.config`. Use `brightnessctl -l` to identify the correct device
