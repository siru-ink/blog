+++
title = "Releasing v3 of Mnemosyne Journal"
date = "2025-10-31"
+++
## Updated GUI Is Finally Live

The major version 3 of the Mnemosyne Journal has finally been released. This
update includes an updated GUI using a themed version of tkinter.[^1]

The soruce code, as always, can be found on my
[Git Forge](https://code.siru.ink/siru-ink/mnemosyne-journal).
Executables can be found here:

* [MacOS App Bundle (not codesigned)](tab:https://drive.google.com/file/d/16VF63NbQrxGdcO8wxKmI7ChHR7zUGvhd/view?usp=sharing)
  * MD5: `b5a8c1bc25ef54dda82dfaf60f7eea4a`
* [Windows EXE (not codesigned)](tab:https://drive.google.com/file/d/197Jh4aYJlqzV3yrP48bsBr246NpD_u2k/view?usp=sharing)
  * MD5: `271177c4cbe3177fb048d690adf7bebb`

However, personally I would suggest simply using the Python source as provided
via [PyPI](https://pypi.org/project/mnemosyne-journal/)[^2], and running it
through the [uv](https://docs.astral.sh/uv/) package manager, using the command:

```bash
uvx --from mnemosyne-journal siru-mnemosyne-gui
```

[^1]: Theme source: [GitHub](https://github.com/rdbende/Sun-Valley-ttk-theme)
[^2]: Python Package Index is the authoritative package repository for the
      Python programming language.
