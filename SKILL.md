---
name: music-reaper
description: Download audio from any yt-dlp-supported site, normalize to -14 LUFS, convert to FLAC with rich metadata, and rename to "歌名 - 歌手 [bitrate]". Use when user provides a URL (YouTube, Bilibili, Niconico, etc.) and asks to download audio, rip music, or save a song.
---

# music-reaper

## Quick start

```powershell
# Extract metadata JSON
> yt-dlp --print-json "<URL>" > meta.json

# Parse values (PowerShell)
> $json = Get-Content meta.json -Raw | ConvertFrom-Json
> $title = $json.title -replace ' - .*', ''
> $artist = ($json.title -split ' - ')[1] -replace ' 「.*', ''
> $bitrate = $json.abr -as [int]
> $date = $json.upload_date -replace '(\d{4})(\d{2})(\d{2})', '$1-$2-$3'
> $comment = "Original bitrate: ${bitrate} kbps | Source: $($json.extractor) | Uploader: $($json.uploader) | URL: $($json.webpage_url)"

# Download, normalize, convert to FLAC with rich tags
> yt-dlp -f "bestaudio[abr>0]/bestaudio" -x --audio-format wav -o "temp.wav" "<URL>"
> ffmpeg -i "temp.wav" -af "loudnorm=I=-14:LRA=+1:TP=-1" -ar 44100 -y "temp_norm.wav"
> ffmpeg -i "temp_norm.wav" -c:a flac -metadata title="$title" -metadata artist="$artist" -metadata date="$date" -metadata comment="$comment" -y "${title} - ${artist} [${bitrate}kbps].flac"
> Remove-Item meta.json, temp.wav, temp_norm.wav -Force
```

## Workflow

1. **Extract metadata JSON**:
   ```powershell
   yt-dlp --print-json "<URL>" > meta.json
   ```

2. **Parse metadata** into variables:
   ```powershell
   $json = Get-Content meta.json -Raw | ConvertFrom-Json
   $rawTitle = $json.title
   $title = $rawTitle -replace ' - .*', ''
   $artist = ($rawTitle -split ' - ')[1] -replace ' 「.*', ''
   $bitrate = [math]::Round($json.abr)
   $date = $json.upload_date -replace '(\d{4})(\d{2})(\d{2})', '$1-$2-$3'
   $comment = "Original bitrate: ${bitrate} kbps | Source: $($json.extractor) | Uploader: $($json.uploader) | URL: $($json.webpage_url)"
   ```
   Title parsing assumes `歌名 - 歌手「其他信息」` format. For simple `歌名 - 歌手`, use:
   ```powershell
   $parts = $rawTitle -split ' - '
   $title = $parts[0]
   $artist = $parts[1..$($parts.Length-1)] -join ' - '
   ```

3. **Download audio** as WAV (highest available quality):
   ```powershell
   yt-dlp -f "bestaudio[abr>0]/bestaudio" -x --audio-format wav -o "temp.wav" "<URL>"
   ```

4. **Normalize LUFS** to -14 (broadcast standard):
   ```powershell
   ffmpeg -i "temp.wav" -af "loudnorm=I=-14:LRA=+1:TP=-1" -ar 44100 -y "temp_norm.wav"
   ```

5. **Convert to FLAC** with rich metadata tags:
   ```powershell
   ffmpeg -i "temp_norm.wav" -c:a flac `
     -metadata title="$title" `
     -metadata artist="$artist" `
     -metadata date="$date" `
     -metadata comment="$comment" `
     -y "${title} - ${artist} [${bitrate}kbps].flac"
   ```

6. **Clean up** intermediate files:
   ```powershell
   Remove-Item meta.json, temp.wav, temp_norm.wav -Force
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
