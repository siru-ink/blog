+++
title = "On installing yt-dlp on a linux machine"
date = "2026-01-19"
+++
> [!Warning]
> Due to changes in ownership, it should nowadays also be noted that the `uv`
> project is run by the OpenAI company nowadays. While the project as of now
> (2026-08-13) is still open source, it remains to be seen whether this will
> still remain the case in the future.

## What is `yt-dlp`?

`yt-dlp` allows you to download media from places that do not provide an easy
download button. It does this by exfiltrating the media streaming urls--and
faking being a logged in browser at times--and stores them locally on a
computer. It even integrates with other software to be able to join different
video- and audio-streams locally, enabling it to download the highest quality
available for each individually.

## How can one get `yt-dlp`?

The recommended way, is to download a pre-packaged binary `yt-dlp` executable
directly from [GitHub](https://github.com/yt-dlp/yt-dlp). This includes most of
what is needed in the executable itself and reduces the need for external
dependencies to a minumum--though not to zero.

## What other stuff does one need to get for `yt-dlp`?

If one takes the easy method as presented above, the only external dependencies
required by `yt-dlp` are: a Python3 interpreter, a JavaScript interpreter, and
FFmpeg. The Python3 interpreter is used to run the `yt-dlp` source code itself.
The JavaScript interpreter is used to deal with modern-day JavaScript challenges
on websites. FFmpeg is used to all `yt-dlp` to download separate video and audio
streams, and then recombine them locally--allowing the downloading of highest
quality streams independently of each other.[^1]

### Python Interpreter via uv

The easiest way to install Python3 on modern systems in a post 2025 landscape is
to use [Astral's `uv` utility](https://docs.astral.sh/uv/). While this
technically installs a separate application on the computer first, it makes
one's life so much easier dealing with the multitude of different Python3
versions, that I cannot emphasize it's usefulness enough. The installation is on
a per-user basis and quite simple following
[their published install script](https://docs.astral.sh/uv/#installation). Once
installed, adding a Python3 interpreter to the system is as easy as:

```bash
uv python install
```

This will auto-select a recommended Python3 interpreter, or one can request a
particular version by appending it to the command as:

```bash
uv python install 3.14
```

> [!Note]
> At the time of writing, the Python3.14 interpreter was the newest stable
> Python3 release. If this is being read in the future, then perhaps a newer
> version would be more appropriate, as the Python3 version release cycle is
> rather tight. More information [here](https://devguide.python.org/versions/).

### JavaScript Interpreter: Deno

A simple way to think of this is, the same way we need to install Python or Java
before we can run programs in these respective language, we must also install a
JavaScript interpreter before we can start running JavaScript programs. However,
why we cannot simply reuse a JavaScript interpreter that is already bundled
together with our browser (assuming you are using a browser from this millenium),
is a mystery best left for another day.

To install Deno, we can simply take the command from their
[website](https://deno.com), as:

```bash
curl -fsSL https://deno.land/install.sh | sh
```

This will download a `deno` executable (and a few other things), and put them
into a `.deno` directory on your home path, which for linux based systems is
usually `/home/<username>/`. Then it will prompt you to add this to the `$PATH`
environmental variable, however, I would recommend doing this manually, so that
you actually know where your computer is looking for programs when you ask it to
execute something on the command line. For the [Fish](https://fishshell.com)
shell you do this by adding the following line to your
`~/.config/fish/config.fish`:

```fish
fish_add_path --path ~/.deno/bin
```

> [!Note]
> Take note of the fact, that we always must add the directory that holds
> executables to the path, and not the executable itself. This seems like a bit
> of a security risk, as it would now be possible to simply copy something into
> this directory to make it accessible for the user everywhere by accident, but
> at least we still have file permissions that prevent things from automatically
> being executable...

### FFmpeg -- the work horse of modern media transcoding

Getting into the details of which FFmpeg version, with which licensing, and
which components compiled into it to use is outside the scope of this post.[^2]
For now, FFmpeg is an old enough project, that it can be found in most package
managers, so it can probably be added by something like `brew install ffmpeg`
or `apt install ffmpeg` etc.

## Quick Intro -- How to use `yt-dlp`

Usage is comparatively simple. One needs a url for a supported web host--
supported web hosts can be listed by the `yt-dlp --list-extractors`. Once such a
url has been obtained, it can simple be plugged into the `yt-dlp` command as:

```bash
yt-dlp -f "bestvideo+bestaudio" "<url>"
```

Which will then attempt to download the video linked at the url--or even
download the entire playlist if a playlist can be identified. Give it a try with
a YouTube video.

[^1]: It is sometimes the case that online streaming service providers will
      present a higher quality audio stream, when only audio is being streamed
      than the one which would be found in a media container alongside a video
      stream.

[^2]: When writing this, I was tempted to try to put all the information about
      FFmpeg into here as well, however, then I would likely never get around to
      posting this publically. Instead, if I ever do get around to writing that
      post, it will be linked ~~here~~ (but I have not gotten around to writing
      it yet.)
