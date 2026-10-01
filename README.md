GreyMatter

GreyMatter is a personal music library and installable web app. Import tracks or playlists, organize songs into playlists, and save selected tracks inside GreyMatter for offline listening.

## Backend architecture

GreyMatter's backend was migrated from Node.js to Python with FastAPI. Node.js is no longer required to run or build the app.

## Features

- Import audio from a YouTube video or playlist; unavailable videos are reported while accessible tracks continue importing.
- Search songs by title, artist, or playlist.
- Create playlists, filter songs by playlist, and shuffle the current view.
- Play songs with previous/next controls and a seekable progress bar.
- Cache tracks in GreyMatter for offline playback, or save an MP3 file to the device.
- Install the frontend as a Progressive Web App (PWA) on supported devices.
- Import MP3s from a device into the browser's local GreyMatter library.

## Requirements

- Python 3.11 or later
- FFmpeg, required for converting imported audio to MP3
- `mkcert`, for the local HTTPS development server and iPhone access

The Docker image installs FFmpeg automatically.

## Run locally on macOS

From the repository root, create the Python environment and install dependencies:

```sh
python3 -m venv backend/venv
backend/venv/bin/python -m pip install -r requirements.txt
```

Install FFmpeg and `mkcert` using your preferred package manager. Then install the local certificate authority once:

```sh
mkcert -install
```

Start GreyMatter:

```sh
cd backend
./run_https.sh
```

Open <https://localhost:8000> on the Mac. The launcher also prints the Mac's current Wi-Fi address for other devices on the same network.

### Open GreyMatter on an iPhone

1. Make sure the iPhone and Mac are on a network that allows devices to connect to each other.
2. Install the Mac's `mkcert` root certificate on the iPhone. Find the certificate directory with `mkcert -CAROOT`, transfer `rootCA.pem` to the iPhone, and install it as a profile.
3. In **Settings → General → About → Certificate Trust Settings**, enable full trust for the certificate.
4. In Safari, open the HTTPS address printed by `run_https.sh`. To install GreyMatter, choose **Share → Add to Home Screen**.

The local certificate is intended for your own devices. Do not share the certificate's private key.

## Run with Docker

Build the image:

```sh
docker build -t greymatter .
```

Create persistent data locations and start the container:

```sh
mkdir -p data/music data/artwork
touch data/music.db
docker run --rm -p 8000:8000 \
  -v "$PWD/data/music:/app/music" \
  -v "$PWD/data/artwork:/app/artwork" \
  -v "$PWD/data/music.db:/app/backend/music.db" \
  greymatter
```

The container serves HTTP on port 8000. For iPhone PWA installation or use outside localhost, put it behind HTTPS. Persist the mounted music, artwork, and database paths so imports survive container replacement.

## Offline use

The installed app caches its interface and saved library information on the device. Tracks streamed from the server are not automatically available offline: open a song's three-line actions menu and choose **Available offline** while GreyMatter is online. Songs imported from device Files are stored in that browser's local storage and cache.

Offline playback requires the song to have been cached in GreyMatter (or imported from Files). If using the hosted Mac server, the Mac must be on and reachable for uncached tracks and imports.

## Data and privacy

Imported audio, artwork, and the SQLite database are stored locally in `music/`, `artwork/`, and `backend/music.db`. These folders and files are excluded from Git; back them up separately if you want to preserve your library.

GreyMatter does not include user authentication. Do not expose an internet-accessible instance without adding an HTTPS reverse proxy and an access-control layer.

## Project layout

```text
backend/     FastAPI server, SQLite database, and YouTube importer
frontend/    HTML, CSS, JavaScript, and service worker
music/       Imported MP3 files (local data, ignored by Git)
artwork/     Imported artwork (local data, ignored by Git)
