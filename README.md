# Stelly

Builds of Stelly, for Linux, Windows and Android, and of its server.

Stelly turns your music into a map where similar tracks sit close together.
From any track you can get its nearest neighbours, a radio that drifts from one
to the next, or a path from one track to another. It plays from Qobuz, so
listening needs a Qobuz subscription.

There are two parts. The **server** runs on one machine and does the heavy
work. The **app** runs on your computers and phone and connects to it.

## Downloads

Everything is on the [latest release](../../releases/latest).

| You have | Download |
|---|---|
| Ubuntu 24.04 | `stelly_<version>_amd64.ubuntu-24.04.deb` |
| Ubuntu 26.04 | `stelly_<version>_amd64.ubuntu-26.04.deb` |
| Another Linux | `stelly_<version>_ubuntu-24.04.AppImage` |
| Windows 10 or 11 | `stelly_<version>_x64-setup.exe` (or the `.msi`) |
| Android 8 or later, 64-bit | `stelly_<version>_arm64.apk` |
| The server, on Ubuntu | `stelly-server_<version>_amd64.<ubuntu>.deb` |
| The server, anywhere else | the Docker image `stellymusic/stelly` |

The `.tar.gz` files contain the same programs, without an installer.

## Installing the app

**Ubuntu**

```sh
sudo apt install ./stelly_<version>_amd64.ubuntu-24.04.deb
```

Use `apt` rather than `dpkg -i`: `apt` also installs the dependencies.

**Other Linux**

```sh
chmod +x stelly_<version>_ubuntu-24.04.AppImage
./stelly_<version>_ubuntu-24.04.AppImage
```

**Windows**

Run the setup `.exe`. Windows may warn about an unrecognised app the first
time: choose *More info* → *Run anyway*.

**Android**

Open the `.apk` on the phone. Android asks to allow installs from your browser
or file manager the first time, then installs it.

## Running the server

With Docker:

```yaml
# compose.yaml
services:
  server:
    image: stellymusic/stelly:v0.17.2   # pin a release, see the tags
    command: ["serve", "--bind", "0.0.0.0:7700"]
    ports:
      - "127.0.0.1:7700:7700"
    env_file:
      - path: .env
        required: false
    volumes:
      - data:/data     # your library, the map, device tokens: back this up
      - cache:/cache   # temporary, safe to delete
    init: true
    restart: unless-stopped

volumes:
  data:
  cache:
```

```sh
docker compose run --rm server pair --name laptop --scope admin
docker compose up -d
```

Or from the `.deb`:

```sh
sudo apt install ./stelly-server_<version>_amd64.ubuntu-24.04.deb
stelly-server pair --name laptop --scope admin
stelly-server serve
```

The server listens on `127.0.0.1:7700` only. It speaks plain HTTP, so to use
it from other devices put it behind a VPN such as WireGuard or Tailscale, or
behind a reverse proxy with TLS. Don't open the port to the internet directly.

The models it uses (about 620 MB) download the first time they are needed.

## Connecting a device

`pair` prints a token once. Open the app, enter the server's address (for
example `http://192.168.1.10:7700`) and that token, and you're in. Pair each
device separately, so that one of them can be revoked later without the others.

The scope decides what a device may do. `admin` can also run the analysis and
manage people and devices; leave it out for a device that only listens.

Each person has their own likes and queue. Add someone with
`user add <name>`, then pair their devices with `pair --name <device> --user <name>`.

## Updates

The apps check this page every few hours. When a new release is out they offer
to update and install it themselves, on Linux, Windows and Android.

The server doesn't update itself: change the image tag and run
`docker compose pull && docker compose up -d`, or install the new `.deb`.
