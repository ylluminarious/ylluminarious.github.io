---

title: How to fix the Emacs Mac Port for multi-tty access
date: 2019-05-23 20:37 UTC
tags:

---


Introduction to the Emacs Mac Port
----------------------------------

[The Emacs Mac
Port](https://bitbucket.org/mituharu/emacs-mac/src/master/README-mac) is an
excellent distribution of Emacs which greatly improves Emacs' functionality on
macOS. It adds native GUI support which provides a slew of nice features. It's
developed and maintained by Mitsuharu Yamamoto and it regularly pulls changes
from GNU Emacs proper, thus staying up-to-date with it. [I've written a fuller
article here](/2019/05/23/emacs-mac-port-introduction/) which provides a general
introduction to the Emacs Mac Port and why I like it.

The bug
-------

Unfortunately, one significant problem I came across when I began to use this
fork of Emacs was its lack of support for [multi-tty
access.](https://www.emacswiki.org/emacs/MultiTTYSupport) This feature is very
useful when connecting to my computer via SSH because it allows me to access my
running Emacs session. In other words, multi-tty access lets me SSH into my
computer and access all my files and buffers in an Emacs terminal frame. FYI,
for those unfamiliar with Emacs, ["frame" is parlance for what is normally
called a
window.](http://www.jesshamrick.com/2012/09/10/absolute-beginners-guide-to-emacs/)
I don't know how many other people make use of this feature, but I've found it
extremely nifty for a long time.

Once I got used to this feature, not having it in the Emacs Mac Port became
vexing. So, I submitted a pull request which added support for this feature. [My
pull request can be seen
here.](https://bitbucket.org/mituharu/emacs-mac/pull-requests/2/add-multi-tty-support-to-be-on-par-with/diff)
I would like to summarize each patch in my pull request to document them for my
own sake and for others' enrichment, in case anyone wants to help work on this.

How I fixed the bug
-------------------

1.  [The first
    patch](https://bitbucket.org/mituharu/emacs-mac/pull-requests/2/add-multi-tty-support-to-be-on-par-with/diff#chg-lisp/server.el)
    involved Lisp code in `server.el` (specifically in `server-process-filter`)
    which stopped multi-tty from working on both Windows and the Emacs Mac Port.
    I removed the code which stopped it from working on the Emacs Mac Port. This
    code would have only served to block the underlying fix from working.

2.  [The second
    patch](https://bitbucket.org/mituharu/emacs-mac/pull-requests/2/add-multi-tty-support-to-be-on-par-with/diff#chg-src/frame.c)
    involved `#ifdef` statements in `frame.c` (specifically in
    `make-terminal-frame`) which essentially did the same thing in C (but at
    compile-time instead of run-time).

3.  [The final
    patch](https://bitbucket.org/mituharu/emacs-mac/pull-requests/2/add-multi-tty-support-to-be-on-par-with/diff#chg-src/macterm.c)
    was the one which actually fixed the problem in `macterm.c` (specifically in
    `mac_mouse_position`). This function was in charge of returning the current
    position of the mouse. The way it was written caused any attempt at
    multi-tty access to crash the whole Emacs server, because the bug occurred
    at the C level. You see, since the Emacs Mac Port expects to be working with
    either graphical frames or terminal frames (but not both), it did not know
    how to get the mouse position from the terminal frame because it expected to
    be working with graphical frames only. In order to fix the problem, all I
    had to do was make a variable called `sf` which would yield the selected
    frame and then make sure that when attempting to fetch the mouse position
    from a given frame, it would ensure that the selected frame was not a
    terminal frame.

And that's all there was to it! I have been happily using multi-tty access in
the Emacs Mac Port by applying these patches and compiling Emacs myself.

Please note that I've only been using this fix on Emacs 26.1, Mac Port 7.4. I
don't yet know whether it will work on 26.2, but I'll update the article when I
get around to it.

**UPDATE:** I've now tested this fix on Emacs 26.2, Mac Port 7.6 which is the
most recent stable version at this time. It still works fine. Note to self:
[read this article](/2019/06/13/how-i-solved-a-cryptic-emacs-dumping-error/) if
you have compilation errors which say something like:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
unexec: not enough room for load commands for new __DATA segments (increase headerpad_extra in configure.in to at least 1488)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For now, if you'd like to apply this fix yourself, try the following
instructions:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ git clone https://bitbucket.org/mituharu/emacs-mac.git
$ cd emacs-mac
$ git checkout emacs-26.2-mac-7.6
$ wget https://bitbucket.org/ylluminarious/emacs-mac-multi-tty/commits/6d23e90288d21546ed2551a7d14dec79ccb3dc13/raw -O patch1.diff
$ wget https://bitbucket.org/ylluminarious/emacs-mac-multi-tty/commits/08874543cb806d0c2dc7b6b5c0f5f92836b7671a/raw -O patch2.diff
$ git apply patch1.diff patch2.diff
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Then compile & install Emacs with your favorite configuration options. I
personally prefer this little ditty (simple, but good enough for the Mac Port):

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ ./configure --with-modules --enable-mac-app --prefix=/Applications/Emacs.app/Contents/Resources && make install
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Why hasn't it been merged yet?
------------------------------

Unfortunately, my fix hasn't been merged with the Emacs Mac Port yet [because
there is a bug which exists on both the NS Port and the Emacs Mac
Port](https://lists.gnu.org/archive/html/emacs-devel/2018-01/msg00430.html)
regarding multi-tty. In the words of the NS Port maintainer, Alan Third:

>   [...] if you do this on the NS port **[or on the Mac Port]**:

>    

>   Open GUI instance

>   start server

>   run ‘emacsclient -t’ in the terminal

>   close GUI frame

>    

>   You can’t open a new GUI frame, even though the GUI instance with the server
>   is still running. You can see it in the dock, and access the menus, but I
>   can’t find any way to create a new frame.

Fortunately, [there is a nice package which works around this
bug.](https://github.com/DarwinAwardWinner/mac-pseudo-daemon) But in order for
Mitsuharu Yamamoto to merge my changes into the Emacs Mac Port, he wanted this
package to be merged into GNU Emacs proper rather than merging it into the Mac
Port which could add an extra maintenance burden for him. Therefore, I went to
the emacs-devel mailing list and proposed merging this package into GNU Emacs.
But since then, life has gotten in the way and I have not advanced on this issue
for about a year.

I also had concerns in the past about whether I would need to sign [FSF
copyright
papers](https://www.gnu.org/prep/maintain/html_node/Copyright-Papers.html) in
order to get my work into GNU Emacs. However, if I recall correctly, Richard
Stallman said to me that I could also put my work in the public domain for each
individual contribution I made to Emacs. Still, life plus this concern has kept
me from completing the proper fix for this issue and I wanted to document it in
case I don't work on it for another year. Hopefully this isn't the case, but
there's no harm in dumping my thoughts before I forget what the heck I was
doing.
