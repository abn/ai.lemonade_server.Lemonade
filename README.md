# Lemonade Flatpak

Flatpak build manifest for [Lemonade](https://github.com/lemonade-sdk/lemonade), a lightweight, high-performance local LLM server and desktop application.

## Prerequisites

- `flatpak`
- `flatpak-builder`

## Building and Installing

To build and install the application for the current user (this will also automatically install required runtimes from Flathub):

```bash
flatpak-builder --force-clean --user --install --install-deps-from=flathub --ccache build-dir ai.lemonade_server.Lemonade.yaml
```

## Running

```bash
flatpak run ai.lemonade_server.Lemonade
```

### Advanced Usage

You can run the server separately from the UI by using the `--command` override:

```bash
# Start only the server
flatpak run ai.lemonade_server.Lemonade --command=lemonade-server serve
```

### System Tray & Background Mode

The Flatpak now includes full support for the Linux system tray (via `libayatana-appindicator`). 
- **Persistence:** By default, the server will continue to run in the background even after the main UI window is closed. 
- **Tray Icon:** You can manage the server, change models, and view status directly from the system tray.

### Running as a Service (Systemd)

To run the Lemonade server automatically in the background on login, you can create a user-level systemd unit file.

1. Create the directory if it doesn't exist:
   ```bash
   mkdir -p ~/.config/systemd/user/
   ```

2. Create `~/.config/systemd/user/lemonade-server.service` with the following content:
   ```ini
   [Unit]
   Description=Lemonade Server (Flatpak)
   After=network.target

   [Service]
   Type=simple
   # Use --command=lemonade-server to start the backend directly
   ExecStart=/usr/bin/flatpak run ai.lemonade_server.Lemonade --command=lemonade-server serve
   Restart=always

   [Install]
   WantedBy=default.target
   ```

3. Enable and start the service:
   ```bash
   systemctl --user daemon-reload
   systemctl --user enable --now lemonade-server.service
   ```

4. Check the status:
   ```bash
   systemctl --user status lemonade-server.service
   ```

### Configuration via Environment Variables

The server supports several [environment variables for configuration](https://lemonade-server.ai/docs/server/lemonade-server-cli/#environment-variables):

- `LEMONADE_PORT`: Port for the server (default: 8000)
- `LEMONADE_HOST`: Binding host (default: localhost)
- `LEMONADE_LOG_LEVEL`: Logging verbosity (critical, error, warning, info, debug, trace)
- `LEMONADE_LLAMACPP`: LlamaCpp backend (vulkan, rocm, cpu)
- `LEMONADE_SDCPP`: SD.cpp backend (rocm, cpu)
- `LEMONADE_CTX_SIZE`: Default context size (default: 4096)

## Maintenance

### Updating `generated-sources.json`

`generated-sources.json` contains the list of all NPM dependencies required for an offline build of the Electron application. It must be updated whenever `package-lock.json` in the main Lemonade repository changes.

To update it:

1. Install [`flatpak-node-generator`](https://github.com/flatpak/flatpak-builder-tools/blob/master/node/README.md) (available via `pipx install git+https://github.com/flatpak/flatpak-builder-tools.git#subdirectory=node`).
2. Obtain the `package-lock.json` from the target version of the Lemonade repository (e.g., `src/app/package-lock.json`).
3. Run the generator:

```bash
flatpak-node-generator npm package-lock.json -o generated-sources.json --electron-node-headers
```

**Note:** Always use the `--electron-node-headers` flag to ensure that native modules can be built correctly inside the sandbox.

