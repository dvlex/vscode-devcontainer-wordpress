# vscode-devcontainer-wordpress

This repository contains a ready-to-use WordPress development environment powered by VS Code Dev Containers.
It starts WordPress and MariaDB automatically and makes the site available at http://localhost:8080.

## Requirements

Before you begin, install these tools:

- [Visual Studio Code](https://code.visualstudio.com/)
- [Docker](https://www.docker.com/products/docker-desktop/)
- [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

## Start The Environment

1. Open this folder in VS Code.
2. Install the Dev Containers extension if you do not have it already.
3. Run **Dev Containers: Reopen in Container** from the command palette, or use **Dev Containers: Rebuild Container** after making changes to the container configuration.
4. Wait for the container to finish building and for the WordPress setup script to complete.

Once the container is ready, WordPress should be running at:

- http://localhost:8080
- http://localhost:8080/wp-admin

## WordPress Setup

WordPress is installed automatically when the devcontainer starts.
The initial site settings, admin user, and plugin list are defined in `.devcontainer/wp-setup.sh`.

By default, this environment is configured for plugin development.
If you want to work on a theme instead, change the mounted volume in `.devcontainer/docker-compose.yml`.

## Reset Or Reload Data

The `Makefile` includes a `reload` target that runs the WordPress setup script again:

```bash
make reload
```

Use it when you want to reset the local WordPress installation or re-apply your setup changes.

Any `.sql` file placed in `.devcontainer/data` is imported during setup with `wp db import`, so you can seed the site with your own data.

Files or folders placed in `.devcontainer/data/plugins` are copied into the WordPress plugins directory and activated automatically.

## Included Tools

- Xdebug
- WP-CLI
- Composer
- Node.js
- MariaDB client tools
- PHP and WordPress VS Code extensions

## Notes

- The environment is designed for local development, so it is safe to rebuild the container whenever you change the devcontainer configuration.
- If you update `.devcontainer/wp-setup.sh`, run `make reload` to apply the changes without rebuilding the whole container.

## TODO

- provide a preconfigured launch.json for PHP debugging
- theme auto-install