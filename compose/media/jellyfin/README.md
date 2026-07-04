# Jellyfin

Standalone Jellyfin deployment that consumes the completed-media output from `private-torrent-downloader`.

## Files

- `docker-compose.yml`: self-contained Jellyfin service definition.

## Usage

1. Create the local bind-mount directories:

   ```bash
   mkdir -p data/config data/cache
   ```

2. If needed, edit `docker-compose.yml` and change `/home/rhew/media/downloads/complete` to the downloader's completed media directory.
3. Start Jellyfin:

   ```bash
   docker compose up -d
   ```

4. Open `http://lenny:8096` or `http://<lenny-ip>:8096`.
5. In Jellyfin, finish the first-run setup and add `/media` as a library path.
6. Enable Intel hardware transcoding in the admin dashboard:

   ```text
   Dashboard -> Playback -> Transcoding
   ```

   Enable hardware acceleration and select the Intel VA-API / Quick Sync option that Jellyfin offers for `/dev/dri`.

## Notes

- This uses host networking so Jellyfin can provide DLNA discovery on the local network.
- Intel Quick Sync transcoding is exposed by mounting `/dev/dri` and adding the host `video` and `render` groups used on `lenny`.
- The media mount is read-only because Jellyfin only needs to index and stream the downloader output.
- If you move this into another repo, copy the whole `jellyfin/` directory and adjust the media path or ports as needed.
