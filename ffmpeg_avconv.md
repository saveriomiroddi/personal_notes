# FFmpeg/Avconv

- [FFmpeg/Avconv](#ffmpegavconv)
  - [Conversion](#conversion)
    - [Audio file](#audio-file)
    - [Video+audio](#videoaudio)
    - [Video to animated GIF/PNG](#video-to-animated-gifpng)
  - [Stream/file operations](#streamfile-operations)
    - [Copy/demux](#copydemux)
    - [Lossless split (trim) m4a](#lossless-split-trim-m4a)
    - [Split by duration, starting on a keyframe](#split-by-duration-starting-on-a-keyframe)
    - [Concatenate videos](#concatenate-videos)
  - [Scaling](#scaling)
    - [Downscale/downmix audio file](#downscaledownmix-audio-file)
    - [Video scaling](#video-scaling)
  - [Other operations](#other-operations)
    - [Check file encoding formats](#check-file-encoding-formats)
    - [Record desktop](#record-desktop)
    - [Build FFmpeg with libfdk-aac support](#build-ffmpeg-with-libfdk-aac-support)

## Conversion

### Audio file

```sh
ffmpeg -i <input> <output.wav>
ffmpeg -i <input> -ab 320k output.mp3
```

### Video+audio

See https://trac.ffmpeg.org/wiki/Encode/MPEG-4.

```sh
# - Default video encoder: libxvid
# - qscale:v = constant quantization, 1-31 (3 -> cartoon/93'/720p ≈ 2.1GiB; 4 -> cartoon/93'/720p ≈ 1.6GiB)
# - qscale:a = VBR, 0-9, (4 ≈ 165 kbps, 5 ≈ 130 kbps)
#
ffmpeg -i input.mp4 -c:v mpeg4 -vtag xvid -qscale:v 3 -c:a libmp3lame -qscale:a 4 output.avi

# CBR Video/Audio (1000k/93' = 838 MiB), and mix both audio channels into two channels (https://trac.ffmpeg.org/wiki/AudioChannelManipulation)
#
ffmpeg -i input.mp4 -c:v mpeg4 -vtag xvid -b:v 1000k -c:a libmp3lame -b:a 96k -af 'pan=stereo|c0<c0+c1|c1<c0+c1' output.avi
```

### Video to animated GIF/PNG

See concatenation section to use multiple input videos as source.

Note that asciinema is nice, however, more complex usages are not supported (eg. switching terminal); in such cases, recording with Virtualbox and converting with ffmpeg is an acceptable strategy.

2 FPS are the bare minimum, for resource-restricted animations. For detail-oriented ones, 4+ are probably a starting point.

GIF:

```sh
# Source: https://superuser.com/a/556031
#
# - `fps=10`: 10 FPS
# - keep the same resolution
# - `-loop -1`: don't loop
#
ffmpeg -i "$input" -vf "fps=10,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop -1 "${input%.*}.gif"
```

(A)PNG; considerably larger than GIF; may not make a noticeable difference.

```sh
# `-plays 10`: 10 loops (use `0` for infinite)
# `-r 1/2`: capture 2 frames every second (keeps the video time the same!)
#
ffmpeg -i "$input" -plays 10 -r 1/2 "${input%.*}.apng"
```

## Stream/file operations

### Copy/demux

```sh
# `c:a copy`:    codec:audio copy
# `-acodec copy: same as above`
# `vn`:          no video; required!
#
ffmpeg -i $input -vn -acodec copy $output

# Demux a single stream.
# See `Stream` from ffprobe (`Stream #0:1: Audio` -> `0:a:1`)
#
ffmpeg -i $input -vn -map 0:a:1 -c copy $audio_output
```

### Lossless split (trim) m4a

See http://uber-rob.co.uk/2014/02/splittingcutting-an-m4a-file.

```sh
# Add `-vn` if there is video
# `-t` is the length
#
ffmpeg -ss 0:00:42 -i in.m4a -c copy -t 1:00:00 out.m4a
```

### Split by duration, starting on a keyframe

```sh
# Cute but underwhelming results (see https://unix.stackexchange.com/q/1670).
#
ffmpeg -i input.mp4 -segment_time 00:10:00 -reset_timestamps 1 -f segment output%02d.avi
```

### Concatenate videos

Source: https://stackoverflow.com/a/11175851

Concat demuxer (lossless): all files must have the same parameters.

```sh
ffmpeg -f concat -safe 0 -i /dev/stdin output.apng << LIST
file '$PWD/01.webm'
file '$PWD/02.webm'
file '$PWD/03.webm'
LIST

# Alternative approach, with wildcard (simplified; doesn't support filenames with single quotes).
#
(for f in *.webm; do echo "file '$PWD/$f'"; done) | ffmpeg -f concat -safe 0 -i /dev/stdin output.apng
```

## Scaling

### Downscale/downmix audio file

```sh
ffmpeg -i $input -sample_fmt s16 -ar 44100 $output    # downscale to 16 bit/44 KHz
ffmpeg -i $input -ac 2 $output                        # downmix to stereo
```

### Video scaling

```sh
# Scale to fit within the specified dimensions with the same aspect ratio, eg. 1280x720 -> 800*450
# (https://trac.ffmpeg.org/wiki/Scaling).
# .
#
ffmpeg -i input.mp4 -vf scale=w=800:h=600:force_original_aspect_ratio=decrease output.avi

# Scale and enforce an aspect ratio
#
ffmpeg -i input.mp4 -vf scale=800:600 -aspect 4:3 output.avi
```

## Other operations

### Check file encoding formats

```sh
ffmpeg -i $filename
ffprobe $filename
```

### Record desktop

```sh
# "-f x11grab -s 1920x1200 -r 10 -i :0.0" = input part; r=FPS
# "-vf scale=1280x800" = scaling
# "-c:v libx264 -preset slower -crf 32" = encoding; crf=constant rate factor
#
# ~2 MB/min
#
# source: https://askubuntu.com/questions/227464/record-my-desktop-in-mp4-format
#
ffmpeg -f x11grab -s 1920x1080 -r 10 -i :0.0 -vf scale=1280x800 -c:v libx264 -preset slower -crf 32 $HOME/Desktop/desktop_recording.mp4
```

### Build FFmpeg with libfdk-aac support

```sh
sudo apt install libfdk-aac-dev

git clone https://github.com/FFmpeg/FFmpeg.git

cd FFmpeg

# Optional (choose the version).
#
git checkout origin/release/4.2

# `-lpthread`: include pthread library
# `-lm`: include standard C math library `libm` (see https://stackoverflow.com/q/1033898)
# --enable-gpl --enable-nonfree: required in order to include also GPL-licensed stuff
#
./configure \
  --extra-libs="-lpthread -lm" \
  --enable-gpl --enable-nonfree --enable-libfdk-aac

make -j $(nproc)

ls -lh ffmpeg
```

Check ffmpeg binary linked libraries:

```sh
ldd `which ffmpeg`
```
