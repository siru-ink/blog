+++
title = "On Creating Icons for Python Applications Using PyInstaller"
date = "2025-10-30"
+++
## The Basics

[PyInstaller](https://pyinstaller.org/en/stable/) is able to take in Python3
packages and convert them into standalone executables of the `.app` or `.exe`
flavor for Windows and MacOS. Today, I would like to discuss some extra
information about its `--icon` command line option. It allows the embedding of a
logo to be used as the app's icon in things like the file explorer, the dock on
MacOS, or the taskbar on Windows. As such, it is not an entirely necessary
addition to an app, but it makes the app seem more polished to an uninformed/end
user. However, the file that can be specified as the command line option differs
slightly depending on whether one is creating an app bundle or an executable.
This will be explained in the following two sections.

## Icns for MacOS

Apple uses the
[`.icns` icon format](https://en.wikipedia.org/wiki/Apple_Icon_Image_format).
This is effectively a directory of different sizes of the same image that the
operating system can choose from to create the best visual experience for the
end user. In order to create such a file one must first create a directory (e.g.
MyIcon.iconset), and then populate the directory with the necessary images.

> [!Note]
> The directory name must end with `.iconset` or the second step will not work.

The images needed inside the directory are as follows:[^1]

{% <box> %}
| Filename            | Resolution |
| ------------------- | ---------- |
| icon_16x16.png      | 16x16      |
| icon_16x16@2x.png   | 32x32      |
| icon_32x32.png      | 32x32      |
| icon_32x32@2x.png   | 64x64      |
| icon_128x128.png    | 128x128    |
| icon_128x128@2x.png | 256x256    |
| icon_256x256.png    | 256x256    |
| icon_256x256@2x.png | 512x512    |
| icon_512x512.png    | 512x512    |
| icon_512x512@2x.png | 1024x1024  |
{% </box> %}

> [!Note]
> For reasons unclear to me, MacOS does not in fact requrie a 64x64 icon size.

After these files have been provided in the `MyIcon.iconset` directory, then on MacOS, the following command can be used to turn this directory into a single `.icns` file instead:

```bash
iconutil -c icns "MyIcon.iconset"
```

This `.icns` file can then be included in the PyInstaller build using the afforementioned `--icon` [CLI](https://en.wikipedia.org/wiki/Command-line_interface) option

## Ico for Windows

For Windows, the process is mostly similar, however it obviously does not use the Apple icns file format. Instead it uses the [`.ico` file format](https://en.wikipedia.org/wiki/ICO_(file_format)). Using the, in my humble opinion, great CLI image editing program [Imagemagick](https://imagemagick.org/), one can create the ico file, including the necessary resized sub-images, with the following command:[^2]

```bash
magick.exe input.png ^
  -define icon:auto-resize=16,32,48,64,256 ^
  -compress zip ^
  output.ico
```

As before, this ico file can then be included in the PyInstaller build using the `--icon` CLI option.

[^1]: For more info see: [StackOverflow](https://stackoverflow.com/questions/12306223/)
[^2]: For more info see: [StackOverflow](https://stackoverflow.com/questions/11423711/)
