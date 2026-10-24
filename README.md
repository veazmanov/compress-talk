# compress-talk

Shrink a recorded slide-show talk into a small MP4 that streams well from a cheap server.

A 15-minute slide talk exported from Final Cut Pro comes out at around **10 MB**.

```bash
compress-talk my-talk.mov
```

## Why it's so small

Talks with slides are an easy case for video compression:

- **Slides barely change.** H.264 only stores what changes between frames, so a slide that stays up for two minutes costs almost nothing. `-tune stillimage` tells the encoder to expect exactly this and keeps text sharp.
- **Speech doesn't need stereo.** One voice is mono anyway; mono AAC at 64 kbps sounds identical for speech and is a fraction of the size.
- **30 fps is plenty.** Screen recordings and phone footage often come in at 60 fps; halving it loses nothing for slides.

## Why it streams well

- **MP4 with H.264 + AAC** plays natively in every browser — no player library, no transcoding on the server.
- **Faststart** moves the file's index to the front, so playback starts immediately instead of after the whole file has downloaded.
- **Capped bitrate** (2 Mbps by default) keeps occasional spikes, like a slide transition with animation, from stalling viewers on slow connections.

The server just serves a static file. Any web server that supports HTTP range requests (Caddy, nginx, Apache — all of them, by default) gives you seeking for free, even on the smallest VPS.

## Install

You need [ffmpeg](https://ffmpeg.org/) with libx264:

```bash
brew install ffmpeg        # macOS
sudo apt install ffmpeg    # Debian/Ubuntu
```

Then put the script somewhere on your `PATH`:

```bash
git clone https://github.com/<you>/compress-talk.git
cd compress-talk
chmod +x compress-talk
ln -s "$PWD/compress-talk" ~/.local/bin/compress-talk
```

## Usage

```
compress-talk [options] INPUT [OUTPUT]
```

If `OUTPUT` is omitted, the result is written next to the input with a `.mp4` extension (or `-compressed.mp4` if the input is already an MP4). The input is never overwritten.

| Option       | Default | What it does                                          |
|--------------|---------|-------------------------------------------------------|
| `-f FPS`     | `30`    | Output frame rate                                     |
| `-h HEIGHT`  | `1080`  | Max height in pixels; smaller videos aren't upscaled  |
| `-q CRF`     | `22`    | Quality, lower is better (18–28 is sensible)          |
| `-a BITRATE` | `64k`   | Mono audio bitrate                                    |
| `-m MAXRATE` | `2M`    | Maximum video bitrate (whole numbers, e.g. `3M`, `800k`) |
| `-p PRESET`  | `slow`  | x264 preset; `fast` encodes quicker, slightly larger  |
| `-y`         |         | Overwrite `OUTPUT` without asking                     |

Examples:

```bash
# Default settings
compress-talk talk.mov

# Lecture with demos or a speaker inset: allow more bitrate
compress-talk -m 4M talk.mov

# Tiny file for slides only, 720p
compress-talk -h 720 -f 15 talk.mov talk-small.mp4
```

## Embedding

```html
<video controls preload="metadata" src="https://example.com/talks/my-talk.mp4"></video>
```

`preload="metadata"` only loads the duration and first frame until someone presses play, which saves bandwidth.

## When not to use it

If your talk has a lot of motion — live demos, filmed footage, a camera on the speaker the whole time — the defaults will still work, but raise the bitrate cap (`-m 4M` or more) and expect bigger files. For long videos with many viewers on poor connections, adaptive streaming (HLS) is the better tool.

## License

MIT
