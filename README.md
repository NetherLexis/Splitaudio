# splitaudio

```
░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
░░      ░░░       ░░░  ░░░░░░░░        ░░        ░░░      ░░░  ░░░░  ░░       ░░░        ░░░      ░░
▒  ▒▒▒▒▒▒▒▒  ▒▒▒▒  ▒▒  ▒▒▒▒▒▒▒▒▒▒▒  ▒▒▒▒▒▒▒▒  ▒▒▒▒▒  ▒▒▒▒  ▒▒  ▒▒▒▒  ▒▒  ▒▒▒▒  ▒▒▒▒▒  ▒▒▒▒▒  ▒▒▒▒  ▒
▓▓      ▓▓▓       ▓▓▓  ▓▓▓▓▓▓▓▓▓▓▓  ▓▓▓▓▓▓▓▓  ▓▓▓▓▓  ▓▓▓▓  ▓▓  ▓▓▓▓  ▓▓  ▓▓▓▓  ▓▓▓▓▓  ▓▓▓▓▓  ▓▓▓▓  ▓
███████  ██  ████████  ███████████  ████████  █████        ██  ████  ██  ████  █████  █████  ████  █
██      ███  ████████        ██        █████  █████  ████  ███      ███       ███        ███      ██
████████████████████████████████████████████████████████████████████████████████████████████████████
one video in, all its parts out
```

one file of bash and ffmpeg that pulls a video apart. the video stream comes back out as its own clean file, and every audio track comes out as a separate mp3. cli if you are in a rush, fullscreen terminal ui if you are not.

```
┌─ splitaudio 1.5 ───────────────────────────────────────────────────────────────────── by NLexis ─┐
│  /tmp/f/demo                                                                                     │
├────────────────────────────────────────────────┬─────────────────────────────────────────────────┤
│  ../                                           │  rocket league ranked.mkv                       │
│  clips/                                        │  ────────────────────────                       │
│▌ rocket league ranked.mkv                 2.0M │                                                 │
│  stream vod.mkv                           2.0M │  duration   0:08                                │
│                                                │  size       2.0M                                │
│                                                │  container  mkv                                 │
│                                                │  video      h264                                │
│                                                │                                                 │
│                                                │  audio tracks  2                                │
│                                                │    0  desktop_audio   aac 1ch                   │
│                                                │    1  micaux   aac 1ch                          │
│                                                │                                                 │
│                                                │  will write                                     │
│                                                │    rocket league ranked_video.mkv               │
│                                                │    rocket league ranked_desktop_audio.mp3       │
│                                                │    rocket league ranked_micaux.mp3              │
│                                                │                                                 │
│                                                │  v1.5 by NLexis                                 │
│                                                │                                                 │
│                                                │                                                 │
│                                                │                                                 │
│                                                │                                                 │
│                                                │                                                 │
├────────────────────────────────────────────────┴─────────────────────────────────────────────────┤
│ ↑↓ move   ⏎ open / split   ⌫ up a folder   t themes   q quit                              NLexis │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## why this exists

you record something with obs, and obs happily dumps your desktop audio, your mic and the game audio into one mkv. now you want the gameplay clean, the commentary separate, also sometimes commentary might be too loud and since its all one track you have to turn down the volume for both, microphone and game/desktop audio. and you do not want to open a video editor or type out six ffmpeg commands with `-map 0:a:1` in them,

this does that in one line.

## what you get

give it `holiday clip.mkv` with two audio tracks and it writes:

```
holiday clip_video.mkv            video only, stream copied, no re-encode
holiday clip_desktop_audio.mp3    named from the track metadata
holiday clip_micaux.mp3           ...or desktop_audio / microphone / track_3 if there is no name
```

track names come out of the container: the track title first, then the handler name obs writes, then a sane fallback. everything gets sanitised so spaces, slashes and emoji do not turn into filesystem arguments.

## install

```bash
git clone https://github.com/NetherLexis/Splitaudio.git
cd Splitaudio
install -Dm755 splitaudio ~/.local/bin/splitaudio
```

needs `bash`, `ffmpeg` and `ffprobe`.

```bash
sudo pacman -S ffmpeg      # arch
sudo apt install ffmpeg    # debian / ubuntu
```

if ffmpeg is missing it says so in one line instead of failing halfway through with some cryptic error.

## how to use

```bash
splitaudio                              # fullscreen tui
splitaudio path/to/some/video.mkv       # split it right now
splitaudio *.mkv                        # batch, one after another
splitaudio theme                        # theme picker (for those who rice and all that)
splitaudio theme gruvbox                # sets it to gruvbox, the picker itself lists every theme there is
splitaudio -v                           # current version
```

outputs land next to the input file. no `-o`, no config to edit, nothing to remember.

## the tui

| key | what it does |
| --- | --- |
| `up arrow` `down arrow` | move through the folder |
| `ENTER KEY` | open a folder, or pick a video and review the plan |
| `backspace button or whatever the hell is called` | goes back (you dont say...) |
| `t` | theme picker, dammit, i made that one other command useless now |
| `q` | quit |

pick a file and it shows you what it found (duration, size, container, codec, each audio track) and exactly which files it is about to write, before it writes anything. hit enter there to send it. then you get a live job list with per-file progress, and a results screen with the sizes when it is done.

## themes

13 of them: catppuccin (mocha, macchiato, frappe, latte), gruvbox, nord, dracula, tokyo-night, rose-pine, kanagawa, everforest, onedark and mono. truecolour if your terminal admits to supporting it, 256 colours if it does not. `NO_COLOR` is respected. your pick is saved to `~/.config/splitaudio/theme`.

## how the split works

```bash
# video stream, straight copy. no quality loss and no encoding wait
ffmpeg -i in.mkv -map 0:v:0 -c:v copy -an in_video.mkv

# one mp3 per track, vbr ~190kbps
ffmpeg -i in.mkv -map 0:a:0 -q:a 2 in_desktop_audio.mp3
```

a 20 minute recording with 2 audio tracks splits in about the time it takes to copy the files, because nothing is encoded except the audio.

the tui is written to keep up with your arrow keys, spaem em all you want, idc: one `ffprobe` per selected file instead of seven, one `du` per folder instead of one per file, and the whole frame is built with `printf -v` instead of a subshell per padded string. it draws a 30 row frame in about 5ms.

## stuff it wont do

worth knowing before you open an issue about it:

- **it does not make your video smaller.** the video stream is copied as is, so the output is the same size and quality as the input. that is the whole point, no re-encode, no wait, no quality loss DID YOU NOT READ OR WHAT?!

- **audio comes out mp3 vbr ~190kbps.** your obs tracks are already aac, so you are not really losing anything worth caring about. if you want wav instead, change the one `-q:a 2` line and you are done

- **nothing gets overwritten, ever.** if `blahblah.mkv` is already sitting there it writes `blahblah_1.mkv` instead. i lost a good funny asf clip to a silent overwrite once and i am still not over it

- **a file with no audio tracks, or no video stream, just gets told that and skipped.** no error wall, no half written bs

- **it writes next to the input**, so that folder has to be writable. it will not ask where you want things to go cuz you already told it by picking the file

- **work on windows**, cuz you know i use arch btw and linux is superior OS (even though its a kernel not an OS please stop calling it an OS you larper), unless by on windows means putting your crappy thinkpad on top of a window then ig? (use WSL on windows)


## license

MIT. do whatever you want with it, just do not sell it... pwease
