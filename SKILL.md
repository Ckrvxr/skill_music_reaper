---
name: music_reaper
description: Download audio from any yt-dlp-supported site, normalize to -14 LUFS, convert to FLAC, and rename to "歌名 - 歌手". Use when user provides a URL (YouTube, Bilibili, Niconico, etc.) and asks to download audio, rip music, or save a song.
---

# music_reaper

## Quick start

```powershell
# Download, normalize, convert to FLAC, rename
> yt-dlp -f "bestaudio[abr>0]/bestaudio" --print "%(abr)s" "<URL>"
> yt-dlp -f "bestaudio[abr>0]/bestaudio" -x --audio-format wav -o "temp.wav" "<URL>"
> ffmpeg -i "temp.wav" -af "loudnorm=I=-14:LRA=+1:TP=-1" -ar 44100 -y "temp_norm.wav"
> ffmpeg -i "temp_norm.wav" -c:a flac -metadata comment="Original bitrate: $bitrate kbps" -y "output.flac"
> Remove-Item "temp.wav", "temp_norm.wav"
```

## Workflow

1. **Get title & artist** — extract metadata for filename:
   ```powershell
   yt-dlp --print "%(title)s" "<URL>"
   ```
   If title contains ` · ` (artists already) or is clearly `歌名 - 歌手`, use it directly.

2. **Get original bitrate** (in kbps):
   ```powershell
   $bitrate = yt-dlp -f "bestaudio[abr>0]/bestaudio" --print "%(abr)s" "<URL>"
   ```

3. **Download audio** as WAV (highest available quality):
   ```powershell
   yt-dlp -f "bestaudio[abr>0]/bestaudio" -x --audio-format wav -o "%(title)s.%(ext)s" "<URL>"
   ```

4. **Normalize LUFS** to -14 (broadcast standard):
   ```powershell
   ffmpeg -i "<file>.wav" -af "loudnorm=I=-14:LRA=+1:TP=-1" -ar 44100 -y "<file>_norm.wav"
   ```

5. **Convert to FLAC** with original bitrate in comment:
   ```powershell
   ffmpeg -i "<file>_norm.wav" -c:a flac -metadata comment="Original bitrate: $bitrate kbps" -y "<file>.flac"
   ```

6. **Rename** to `歌名 - 歌手 [$bitrate`kbps].flac` format (use web search or YouTube description to resolve artist if needed).

7. **Clean up** intermediate files:
   ```powershell
   Remove-Item "<file>.wav", "<file>_norm.wav" -Force
   ```

## Cookies (for login-required sites)

If user provides a cookies file, append to every yt-dlp command:
```powershell
--cookies "<cookie_file_path>"
```

Bilibili always needs `--add-header`:
```powershell
--add-header "Referer:https://www.bilibili.com"
```

## Requirements

- [yt-dlp](https://github.com/yt-dlp/yt-dlp)
- [ffmpeg](https://ffmpeg.org/)
