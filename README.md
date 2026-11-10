# Qbeat

**Multi-channel Discord music playback through a controller bot and a pool of worker bots.**

Qbeat delegates playback to separate worker accounts so multiple voice channels can run independently. Its control panel, queue management and media-provider integrations are documented below.

**Explore:** [Architecture](#why-a-cluster) · [Features](#features) · [Setup](#getting-started) · [Structure](#project-structure)

## Why a cluster

A single Discord bot account can only be in one voice channel per server. To play music in several channels at the same time, Qbeat runs one **main bot** and any number of **worker bots**. The main bot handles slash commands. When someone asks for music, it hands the voice channel to a free worker, and each worker plays in its own channel. If every worker is busy, the user is told to wait.

```
Discord server
  |
  +-- Main bot (DISCORD_TOKEN)        slash commands, assigns channels to workers
  +-- Worker bot 1 (WORKER_TOKENS)    plays in voice channel #1
  +-- Worker bot N                    plays in voice channel #N
```

## Features

- **Interactive control panel.** `/bot-kontrol` opens a panel with buttons for previous, next, pause and resume, stop, shuffle, repeat, the queue, and adding a song or a playlist.
- **Spotify, YouTube and links.** Search by name or paste a YouTube or Spotify link. Spotify playlists are resolved quickly and played from YouTube.
- **Automatic suggestions.** An autoplay option queues related tracks when the queue runs out.
- **No stored audio.** Songs are streamed with `yt-dlp` and FFmpeg and are never saved to disk.
- **Limits you can tune.** Maximum queue size, maximum playlist size and an idle timeout that disconnects an unused bot.
- **Slash commands only.** There is no text prefix.
- **Container ready.** A Dockerfile is included, and the bot forces IPv4 to avoid IPv6 connection problems on Railway.

## Commands

| Command | Purpose |
|---------|---------|
| `/cal <song>` | Start playing a song name or a YouTube or Spotify link in your voice channel |
| `/bot-kontrol` | Open the music control panel |

You must be in a voice channel to use them.

## Getting started

### Requirements

- Python 3.12 or later and [FFmpeg](https://ffmpeg.org)
- A Discord application with a bot token for the main bot, plus one bot token per worker
- Spotify client ID and secret
- A YouTube Data API v3 key

### Run locally

```bash
git clone https://github.com/Burak-ML/Qbeat.git
cd Qbeat/discord-music-bot
pip install -r requirements.txt
cp .env.example .env
python bot.py
```

Fill in `.env`:

```env
DISCORD_TOKEN=main bot token
DISCORD_APPLICATION_ID=main bot application id
WORKER_TOKENS=token1,token2     # comma separated, one per worker bot
SPOTIFY_CLIENT_ID=...
SPOTIFY_CLIENT_SECRET=...
YOUTUBE_API_KEY=...
TEST_GUILD_ID=                  # optional, syncs slash commands to one server instantly
LOG_LEVEL=INFO
IDLE_TIMEOUT_SECONDS=300
MAX_QUEUE_SIZE=500
MAX_PLAYLIST_SIZE=300
```

### Discord permissions

Invite every bot with the `bot` and `applications.commands` scopes and these permissions only. Administrator is not needed.

View Channels, Send Messages, Embed Links, Read Message History, Connect, Speak, Use Application Commands.

### Deploy on Railway

1. Push the repository to GitHub and create a Railway project from it.
2. Add the variables from `.env.example` in the project's **Variables** tab.
3. Railway builds the Dockerfile and starts the bot. Check the logs for the "slash commands synced" message.
4. In Discord, run `/bot-kontrol` to confirm the commands are registered.

## Project structure

```
discord-music-bot/
  bot.py             starts the main bot and the workers
  config.py          environment settings
  cogs/music.py      slash commands and worker assignment
  views/             control panel buttons and the add-song dialog
  services/          YouTube, Spotify, audio, input parsing and recommendations
  models/            track and per-channel player state
  Dockerfile
```

## License

MIT. See [LICENSE](LICENSE).
