# Self-hosting music
Spotify has increased my Premium subscription fee by 27% in the last year. Here's my strategy for self-hosting my own
music streaming service that rivals Spotify and other large platforms. This repository contains the code and configurations
required to do this.

## `docker-compose.yaml`
The `docker-compose.yaml` file contains the configuration required to download and start the docker containers
required to run a self-hosted music streaming platform and ditch Spotify.

### Lidarr
Lidarr is a music collection manager for Usenet and BitTorrent users. Think of it as a personal music curator – helping
to find, download, and organize your favourite artists music automatically! Once set up, Lidarr will monitor your favourite
artists and can fetch new releases or even complete discographies, keeping your collection up-to-date.

Lidarr requires some setup for it to work, refer to their [documentation](https://wiki.servarr.com/lidarr) or there
are plenty of other tutorials online.

**Note:** One tweak once Lidarr is setup. Add **Singles** and **EP's** to your **Standard Metadata Profile**.
By default, Lidarr will only monitor artists albums. This change allows Lidarr to automatically monitor both singles and
EP’s, not just albums.

### Beets
[Beets](https://beets.readthedocs.io/en/stable/index.html#) is another music library management system that uses the
MusicBrainz database to tag music files with appropriate metadata. This includes album names, artist details, release
dates, genres, and album artwork. It will also provide more organization to your music directory by automatically renaming
and sorting music files, while removing any duplicates to make storage more efficient.

Once you’ve downloaded some music files, run the following command using beets:

`docker exec -it beets beet import --group-albums /music`

This will import your music one-by-one for each artist and correct for any missing metadata. Occasionally, this command
requires user input for songs or albums that beets is not 100% confident for. There are 4 options to choose from:

* a: apply the suggested match
* u: use current metadata
* m: manual custom search
* s: skip the current album

**Note:** The `--group-albums` flag is needed to group songs from the same album together for each artist. Without it,
beets will iterate through each artist and only tag one album per artist. Unfortunately, this flag breaks the `incremental`
property of the `import` function, where only newly added music files will be imported on the next run. This is a bug outlined
on the Beets [GitHub](https://github.com/beetbox/beets/issues/1476). This means when new music is added, the newly added
artist must be specified in the `import` command, otherwise the import will re-run on the entire music directory again.

For example, adding new music from "Luke Combs" changes the command to:

`docker exec -it beets beet import --group-albums /music/Luke\ Combs`

After the initial import, run the `import` command occasionally to keep your library up to date. No automation is possible
given that user input is required.

### Navidrome
There are many options to stream your music from the server to client devices. The two main being
[Jellyfin](https://jellyfin.org/docs/) and [Navidrome](https://www.navidrome.org/docs/overview/). This guide uses Navidrome
because it is specialized for music media.

Navidrome is open-source and supports multiple applications for Android, iOS, and Desktop to provide a user-friendly GUI.
A complete list of tested and supported applications are found [here](https://www.navidrome.org/docs/overview/#apps).

I recommend [StreamMusic](https://music.aqzscn.cn/docs/versions/latest/) for mobile devices supporting either Android or iOS,
and [Supersonic](https://github.com/dweymouth/supersonic) music for desktop supporting Windows, Linux, or MacOS. Both options
provide simple interfaces that work and can also display lyrics.

### Podgrab
Podcasts can be easily downloaded and managed using [Podgrab](https://github.com/akhilrex/podgrab). The setup is extremely
simple and easy, and downloaded episodes can be put into the same directory as your music allowing Navidrome to access and
serve the podcast via your chosen streaming application.

## `beets/`
This is the beets configuration directory that contains the `config.yaml` file for the beets docker container. I've added
the `lyrics` plugin so that lyrics are also added to the metadata of music files during imports.

## `Spotify_popular_artists.py`
Use your Spotify streaming history to create a comprehensive list of your favourite artists. These artists can be imported
into Lidarr one-by-one to build a strong music library.

### Instructions
1. Request your data from your Spotify account under ***Security and privacy > Account privacy > Download your data***.
2. Obtain all files named `StreamingHistory_music*.json`. These contain your entire streaming history with Spotify.
3. Run the `Spotify_popular_artists.py` script in the same directory as your streaming history data.
This script will create a list of your most popular artists based on two criterion:
	* (1) number of times you streamed a song from a given artist (variable name `STREAM_CUTOFF`)
	* (2) number of unique songs played from each artist (variable name `UNIQUE_SONG_CUTOFF`)
4. Check the line plots from the `popular_artists.png` and adjust these two criterion accordingly through
the `STREAM_CUTOFF` and `UNIQUE_SONG_CUTOFF` variables in the `Spotify_popular_artists.py` script.
5. Repeat steps 2-4 until the you are satisfied with the number of Spotify artists.

This will result in a `popular_artist.txt` file containing your most popular artists from Spotify.
Each artists from this list can be added in Lidarr to monitor and manage your library of self-hosted music.

## Conclusion
With a bit of work and configuration, you too can take back control of your library of music and ditch those money hungry music
streaming providers. I hope you enjoyed this guide, now you can give it a try!
