+++
title = "Spells for the tty: FFmpeg"
date = "2026-03-23"
+++
As the inaugural post in this _Spells for the tty_ series, I chose to write 
about FFmpeg. Remembering the correct syntax on the command line for FFmpeg has 
always seemed like an impossible feat to me. I am quite thankful for the 
existence of resources such as the 
[H.264](https://trac.ffmpeg.org/wiki/Encode/H.264) 
and [H.265](https://trac.ffmpeg.org/wiki/Encode/H.265) guides. While the guide 
on [AV1](https://trac.ffmpeg.org/wiki/Encode/AV1) has always seemed a bit 
clunky, this is likely due to the rapid development that AV1 has still been 
undergoing.

## So what exactly is _Spells for the tty_ supposed to be?

To put it simply: a cheatsheet for me to reference when I need to use these 
commands in real life myself. However, I sincerely doubt I am the only person 
using these, so to others reading this, perhaps it can be a quick overview of 
well known commands as well as some of the lesser known options or odities that 
one can run into while using these commands.

## But syntax is often variable depending on the system?

True, so let me clarify that these short articles will mostly be focussed 
around the Bash shell on a Debian linux system. To clarify this, I will try to 
append `$>` to all commands. If I end up writing about other shells I'll make 
sure to clarify it in the text.[^1]

---

... so onto FFmpeg now:

## How can I make a certain channel the default of it's type?

```bash
$> ... -disposition:v:1 forced  # :v/a/s: for different channel types
```

## How can I get rid of chapter metadata?

```bash
$> ... -map_chapters -1
```

## How can I add a cover image?

```bash
$> ... -i <cover.jpg> ... -c:v:1 mjpeg -disposition:v:1 attached_pic
```

> [!Note]
> In my entirely personal opinion dealing with some of the peculiarities of
> `mp4` files can be dangerous. If you are planning to include a lot of
> different types of metadata into your media files, I would strongly suggest
> taking a look at the `mkv`/Matroska video-file format. It has also has great
> command line support via [MKVToolNix](https://mkvtoolnix.download).

---

[^1]: If you came all the way to this footnote to make fun of me still using bash as my default shell, then I humbly suggest to you to go get a life. Whether it is the best or not, it is realiable in the sense that is available almost everywhere, often times even windows via `git bash`.
