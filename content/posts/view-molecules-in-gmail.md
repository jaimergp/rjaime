---
title: "View Molecules in Gmail & Drive"
date: 2018-08-26T02:19:01+02:00
draft: true
---

TLDR: Drive Molecule Viewer is a Google Drive and Gmail addon to view molecular structures (PDB, Mol2...) in the browser, without having to download the files to disk nor installing anything. Keep reading for configuration and usage instructions.

<!-- image here! -->

<!--more-->

---

**Note**: If you do not care about contextual details, feel free to skip to section [Configuration & Usage](#configuration-usage) right way.

My PhD in Biotechnology had a strong focus on computational chemistry and molecular modeling. As a result, many of my email correspondence consists of attachments like PDB, XYZ or Mol2 files. These file formats contain information to depict 3D molecular structures suitable for interactive visualization. However, viewing them means downloading them to disk and opening with specialized software like:

- UCSF Chimera
- PyMol
- VMD

Most of the time, you only want to have a quick look, though. Since I hate convoluted folders (and *~/Downloads* is one the worst), I created a small Google Drive addon that allows you to see these files in-browser. It relies on the excellent [NGL viewer](http://nglviewer.org/#ngl) and the free [Google Script platform](https://www.google.com/script/start/). Source code is available in [my GitHub profile](https://github.com/jaimergp/drive-molecule-viewer).

# Configuration & Usage

## Configuration

No actual installation is needed. You only need to register the addon within your account.

1. Go to the Drive Molecule Viewer [Google Script macro](https://script.google.com/macros/s/AKfycbwRI6Xnx1MKKTLBGl3_2sKJemME4pMJFB8Gp0FmaFiD/exec). Since I have yet to register the addon in the Google directory, it will first tell you that the script is potentially insecure. Click on... XXXX.

2. Then, it will prompt for permissions to register in your Google Drive profile. It needs full access to your Drive files (I still have to figure out how to make it work with read-only permissions), but it won't touch anything. If you don't trust me, [check the source](https://github.com/jaimergp/drive-molecule-viewer)!

3. After that, the app will be available as an `Open with` entry in your file contextual menus. This includes both Drive files and Gmail attachments! 

![Image here]()

## Standalone usage

The app can also be loaded separately from Google Drive or Gmail. Simply go to [Drive Molecule Viewer](https://script.google.com/macros/s/AKfycbwRI6Xnx1MKKTLBGl3_2sKJemME4pMJFB8Gp0FmaFiD/exec) and click on `File > Import from Google Drive`.

## Open attachments in your smartphone

The Gmail app does not offer an `Open with` menu. As a result, if you want to open a molecule attachment from the app, you have to manually save it to your Drive files. Then, you can go to the Drive app and open the newly created file as usual. The Drive Molecule Viewer will load the file automatically.

# Help & Support

Should you have any doubts, questions or suggestions, feel free to [submit an issue](https://github.com/jaimergp/drive-molecule-viewer/issues) in the GitHub repo. I will post new updates in [this Twitter thread](https://twitter.com/jaime_rgp/status/1025395287672676354).