# Skills

## Install

1. Add the marketplace.

   ```
   /plugin marketplace add Berkeley-Humanoids/Skills
   ```

2. Install plugins.

   ```
   /plugin install berkeley-humanoids-skills@berkeley-humanoids-skills
   ```

## Upgrade an installed version

The plugin tracks this repository, so the marketplace refresh also moves
the installed plugin to the current commit:

1. Update the marketplace.

   ```
   /plugin marketplace update berkeley-humanoids-skills
   ```

2. Restart Claude Code. The new version applies after the restart.

`claude plugin list` prints the installed version of each plugin as a
commit hash.

## License

CC0 1.0 Universal. See [LICENSE](LICENSE).
