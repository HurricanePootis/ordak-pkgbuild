# Why this was made
I created this because the AUR package may or may not be nuked/never updated, so I am creating this PKGBUILD on github for myself and anyone who wants to use it.

# Quality of the PKGBUILD
As of now, I am not going to try and manually do any of the building deps stuff. I'm just going to use as much as I can from the system and then use whatever it gives me. You can see this from my liberal use of patchelf.

# Debugging & Reporting
Now, as anyone reading this may or may not know, the whole reason behind the AUR controversy is due to—at least how I understand it—the fact that the behavior of an official build and an AUR build may be different, especially when it comes to bugs. So, if you are using this PKGBUILD (or really, any other emulator's PKGBUILD), before you go report a bug, please download their emulator from them and use it however they intend it to be used. This could be an AppImage, tar file, or a flatpak.

Now, if the issue cannot be replicated with the official binary, then it is an issue with PKGBUILD. In order to build a debug build, in the `PKGBUILD`, please change `_debug=false` to `_debug=true`. This will build the software and libraries with debug info and will also not strip the package. Using this debug version, one may go through with `gdb` and debug.

But, if the issue can be replicated with the official binary, then please, for the love of god, if you are going to make a report upstream, don't even mention the fact that you used an unofficial build, because it literally does not matter. As long as the bug exists in the upstream builds, then please be courteous and respectful whenever making a bug report, if you choose to do so. Furthermore, please understand that the developers of this software are not obligated to help nor to treat you with any sort of priority or urgency. Please remember that there are human beings behind each github developer, and they have reasons for acting the way they do.

# Glorp
Glorp.
