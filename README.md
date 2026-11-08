<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/readme/hero-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/readme/hero-light.svg">
    <img src="assets/readme/hero-dark.svg" width="880" alt="Spotifybud: a Telegram bot with 4 commands that converts SoundCloud playlists to Spotify, recommends songs and builds 10-track Spotify playlists. Example chat with a /playlist request.">
  </picture>
</p>

<p align="center">
  <strong>A Telegram bot that converts SoundCloud playlists to Spotify, recommends songs, and builds playlists from one search.</strong><br>
  <sub>Python, python-telegram-bot, Spotipy and soundcloud-lib. Bot: <a href="https://t.me/spotifybuddy_bot">@spotifybuddy_bot</a> (it only responds while <code>app.py</code> is running) <!-- verify: bot is still registered --></sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-121212?style=flat-square&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/python--telegram--bot-121212?style=flat-square&logo=telegram&logoColor=26A5E4" alt="python-telegram-bot">
  <img src="https://img.shields.io/badge/Spotify%20Web%20API-121212?style=flat-square&logo=spotify&logoColor=1DB954" alt="Spotify Web API">
</p>

---

Spotifybud does common Spotify tasks from a Telegram chat. Send it a SoundCloud playlist link and it rebuilds the playlist on Spotify. Send it a song and it suggests a similar one. Send it any search term and it makes a 10-track playlist and replies with the link.

## Demo

Convert a SoundCloud playlist:

![Converting a SoundCloud playlist](https://github.com/haashirk20/spotify-bud/assets/114118228/bc62fb9a-1a58-474c-b454-04cd56577935)

Get a recommendation:

![Song recommendation](https://github.com/haashirk20/spotify-bud/assets/114118228/1634e314-bcd3-4645-81bc-bfa2d9372f73)

Generate a playlist:

![Generated playlist](https://github.com/haashirk20/spotify-bud/assets/114118228/163d21cb-33b9-46f4-905b-4244fcaa9136)

## Commands

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/readme/commands-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/readme/commands-light.svg">
    <img src="assets/readme/commands-dark.svg" width="880" alt="Commands: /convert, /recommend, /playlist and /help, with the bot's reply to each.">
  </picture>
</p>

`/recommend` and `/playlist` share one helper. It searches Spotify for your text, takes the top track's ID, and calls the recommendations endpoint with that track as the seed. `/recommend` replies with the first result; `/playlist` puts all 10 into a new public playlist on the bot owner's account.

## Converting a playlist

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/readme/convert-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/readme/convert-light.svg">
    <img src="assets/readme/convert-dark.svg" width="880" alt="Converting a playlist: SoundCloud link, resolve with sclib, search Spotify for each track by artist and title, create a playlist with a 5-character suffix, reply with the link.">
  </picture>
</p>

`sclib` resolves the SoundCloud URL into a playlist and its tracks. For each track the bot searches Spotify for the artist and title and takes the first match. New playlist names get a 5-character suffix from Python's `secrets` module, so the bot can find the playlist it just created among the account's playlists and add the tracks to it.

## Architecture

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/readme/architecture-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/readme/architecture-light.svg">
    <img src="assets/readme/architecture-dark.svg" width="880" alt="Architecture: Telegram chat, lib/app.py command router, lib/spotify_api.py, Spotify Web API and SoundCloud, with credentials in a gitignored lib/auth.py.">
  </picture>
</p>

`lib/app.py` is the Telegram side: one handler takes every command and routes it by name to a function that returns the reply text. `lib/spotify_api.py` holds all the Spotify and SoundCloud calls. Credentials stay in a gitignored `auth.py` module, and the Spotify scope is limited to `playlist-modify-public`.

| Area | Tools |
|---|---|
| Language | Python |
| Bot framework | python-telegram-bot (async `ApplicationBuilder`, long polling) |
| Spotify | Spotipy with the OAuth authorization-code flow |
| SoundCloud | `soundcloud-lib` (`sclib`) |
| Packaging | PyInstaller (Windows `.exe` in `app/`) |

## Getting started

You need a Telegram bot token from [@BotFather](https://t.me/BotFather) and a Spotify app from the [Spotify developer dashboard](https://developer.spotify.com/dashboard) with a redirect URI set.

1. Install the dependencies (there's no `requirements.txt` yet):

   ```sh
   pip install python-telegram-bot spotipy soundcloud-lib
   ```

2. Create `lib/auth.py`:

   ```python
   token = "<telegram bot token>"
   spotify_client_id = "<spotify client id>"
   spotify_client_secret = "<spotify client secret>"
   redirect_uri = "<redirect uri registered for your Spotify app>"
   ```

3. Start the bot:

   ```sh
   cd lib
   python app.py
   ```

   On the first Spotify call, Spotipy opens a browser to authorise your Spotify account and caches the token in `.cache`. Playlists are created on that account.

## Known limitations

- Every playlist is created on the bot owner's Spotify account, not the account of the person chatting.
- Spotify limited access to the recommendations endpoint for new apps in late 2024, so `/recommend` and `/playlist` may not work with a newly created Spotify app. <!-- verify -->
- `/convert` assumes every SoundCloud track has a Spotify match. A track with no search result makes the conversion fail.
