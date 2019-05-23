---

title: Emacs Mac Port Introduction
date: 2019-05-23 20:38 UTC
tags: 

---


Valuable Features & Improvements
--------------------------------

[The Emacs Mac
Port](https://bitbucket.org/mituharu/emacs-mac/src/master/README-mac) is an
excellent distribution of Emacs which greatly improves Emacs' functionality on
macOS. It adds native GUI support which provides a slew of nice features. I
would like to name some of those features which I particularly enjoy, so please
note that this is not a full list of the features and improvements. If you would
like a full list, check out [the README
file](https://bitbucket.org/mituharu/emacs-mac/src/master/README-mac) which is
well worth your time if you use macOS and Emacs. The following items are taken
from there, so again, if it looks like something is missing, then please look at
the full list because these are just *my* favorite features:

-   Better `C-g` handling

-   It doesn't use CPU time while the Lisp interpreter is idle and waiting for
    some events to come, even with subprocesses or network connections.

-   Graceful termination: If you try logout/shutdown/reboot while leaving a
    file-visiting buffer modified and unsaved, a popup window appears for
    confirmation. If you cancel the termination of Emacs (including C-g or ESC),
    the whole logout/shutdown/reboot process is also canceled immediately (i.e.,
    you will see a "canceled" dialog immediately rather than a "timed out" one
    afterward). If you don't have unsaved buffers, shell buffers, etc., you
    won't see unnecessary confirmation.

-   Apple event handling: One can define Apple event handlers at the Lisp level.
    Actually, graceful termination above is an instance of Lisp-level Apple
    event handling. Another example is "Get URL" handler that enables us to
    invoke the mailer you customized with `mail-user-agent`, e.g.,

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ osascript -e 'tell application "Emacs" to open location "mailto:foo@example.com"'
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you set Emacs as the default mailer via Mail.app preference, the Emacs mailer
will set up a draft buffer when you click a `mailto:` link in a Web browser.

-   DictionaryService support: You can look up a word under the mouse pointer in
    the selected window by typing `Command-Control-D` (or double/single-tapping
    a trackpad with three fingers on Mac OS X 10.7/10.8, resp.).

-   Some minor visual enhancements

    -   Aligned key bindings in menus

    -   Progress indicator (corresponding to hourglass) in the title bar

    -   Unusable items in the font panel are hidden

-   The `fullscreen` frame parameter, with all values supported: `fullboth`,
    `fullwidth`, `fullheight`, and `maximized`. The `fullboth` frames, which
    don't have the title bar, still allow us to access the menu bar, the Dock,
    and the tool bars. The menu bar can also be activated via `menu-bar-open`,
    `Control-F2` (if full keyboard access enabled), or `Command-Shift-/` (on Mac
    OS X 10.5 and later) even for fullboth frames where the menu bar is usually
    hidden. Changing fonts or internal-border-width in fullscreen frames does
    not clutter display. On multiple monitor environments, one can move
    fullscreen frames to another monitor by setting the `left` and `top` frame
    parameters accordingly. Attaching/detaching external monitors should work
    even with fullscreen frames.

-   The `sticky` frame parameter, which allows us to keep particular frames
    visible for all Spaces on Mac OS X 10.5 and later.

-   The function `system-move-file-to-trash`, which can be specified as a value
    of `delete-by-moving-to-trash`.

-   SVG image display. This can be done via the WebKit framework, so you don't
    need librsvg.

-   Unicode character display including non-BMP ones.

-   Complex Text Layout and text shaping. They are implemented using the Core
    Text or NS Text layout engine, so you don't need libotf.

-   Can be compiled with the ImageMagick support. Even without the ImageMagick
    library, the Mac port provides a fallback using the Image I/O framework so
    you can scale and rotate images.

-   The variable `tool-bar-style` works like in GTK+. The values `both-horiz`,
    `text-image-horiz` are synonymous with `both`.

-   Horizontal scroll bar support.

-   Pixel-based mouse wheel smooth scroll for newer mice/trackpads.

-   Gesture event handling for newer trackpads. By default, pinch out/in are
    bound to text size scaling. With the shift key, they turn on/off fullscreen
    status of the frame.

-   "Click in the scroll bar to: Jump to the spot that's clicked" setting in the
    System Preferences is supported. Pressing the option key while clicking
    toggles this behavior temporarily.

-   Change of text smoothing threshold setting in the Appearance pane of the
    System Preferences is reflected immediately.

-   Several keyboard shortcuts (notably those for Keyboard Navigation) listed in
    System Preferences just work like other applications.

-   When the clipboard has both textual and image data, yank inserts the former
    and push both into the kill ring so the latter can be inserted with yank-pop
    afterwards.

-   Can display color bitmap fonts such as Apple Color Emoji, if compiled and
    executed on Mac OS X 10.7 or later. Also supports display of some
    combinations of regional indicator symbols, such as U+1F1EF followed by
    U+1F1F5, as national flags. Variation Selectors 15 (text-style) and 16
    (emoji-style) are also supported. On OS X 10.10.3 and later, emoji modifiers
    for skin tones (U+1F3FB - U+1F3FF) are supported as well.

-   New function \`mac-start-animation', which provides animation effects on Mac
    OS 10.5 and later via Core Animation. You can see the default animations
    with buffer switching by horizontal swiping/flicking, exiting from the
    splash screen by typing "q", and the "About Emacs" and "Preferences..." menu
    items in the application menu (labeled "Emacs") in the menu bar.

-   High-resolution image display support for Retina displays. Dynamic
    resolution change is also supported. For bitmap formats, high-resolution
    data can be specified either by the "\@2x" file name convention, the
    ":data-2x" property, or multi-image TIFF. Shrinking a large image with the
    "imagemagick" (or Mac-specific "image-io") image type also works but gives a
    non-optimal result on dynamic resolution change. For vector formats, images
    are automatically re-rendered according to the resolution of the frame.
    DocView mode and LaTeX fragment preview in Org mode are modified so they can
    take advantage of this feature.

At the time of this writing, it is more than just an improvement; [it lacks a
bug](https://lists.gnu.org/archive/html/emacs-devel/2019-03/msg00901.html) in
the official NS Port [which breaks menus on
Mojave.](https://debbugs.gnu.org/cgi/bugreport.cgi?bug=32864) Suffice it to say,
the Emacs Mac Port is my current choice of Emacs distro and the best choice I've
seen for macOS ([Aquamacs is another good one,](http://aquamacs.org/) but
unfortunately it does not integrate with the native macOS GUI and its changes
only exist at the Lisp level).

The only problem it has is lack of multi-tty support, but I managed to fix this
issue [which I documented
here.](/2019/05/23/how-to-fix-the-emacs-mac-port-for-multi-tty-access/)

History & Future
----------------

The Emacs Mac Port is developed and maintained by Mitsuharu Yamamoto and it
regularly pulls changes from GNU Emacs proper, thus staying up-to-date with it.
It exists as a fork because at the time of its initial release, [it wasn't quite
ready](https://lists.gnu.org/archive/html/emacs-devel/2019-03/msg00895.html) so
the Emacs developers chose the so-called NS Port instead (which is now
technically inferior). Why is it called the NS Port? Because Emacs historically
preferred the term
["Nextstep",](https://www.gnu.org/software/emacs/manual/html_node/emacs/Mac-OS-_002f-GNUstep.html)
presumably due to a desire to support
[GNUstep.](https://en.wikipedia.org/wiki/GNUstep) [The maintainer of the NS Port
acknowledged that the Mac Port does a better job in a number of
areas:](https://lists.gnu.org/archive/html/emacs-devel/2019-03/msg00919.html)

>   The Mac port has a better maintainer than me, who knows the toolkit and
>   things better than I could ever hope to. It ties into macOS better (I use
>   very little macOS specific software, and the whole ‘services’ thing is a
>   mystery to me, for example). It doesn’t have the Mojave redraw problems that
>   have me completely scunnered at the moment. It handles concurrency better.
>   It is simply better maintained.

Yet its use of features not supported by GCC might rule it out as a replacement
candidate:

>   But then again, the Mac port makes use of things that gcc doesn’t support,
>   like ObjC blocks, so that would probably rule it out as an official port
>   immediately.

>   The duplication of effort to maintain two ports seems like a waste to me,
>   but there are definitely barriers to joining forces and having only one,
>   official, port.

This could cause Richard Stallman to be against merging the Mac Port in place of
the NS Port [because he wants GNU projects to use
GCC.](https://gcc.gnu.org/ml/gcc/2014-01/msg00247.html) He may also object to it
if he perceives that it supports features not found on Linux, [as he has been
known to raise such issues in the
past.](https://github.com/emacs-mirror/emacs/blob/emacs-25.1/etc/NEWS#L1723-L1730)

I think it is too bad that the Mac Port has not yet replaced the NS Port because
it is superior with its native macOS integration. The developer has clearly
worked hard and his work is praiseworthy. I hope the Emacs developers do not
allow the aforementioned issues to be obstacles which disallow the Mac Port from
merging into the Emacs source code.
