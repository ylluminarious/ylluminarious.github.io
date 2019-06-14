---

title: How I solved a cryptic Emacs dumping error
date: 2019-06-13 19:00 UTC
tags:

---


Background
----------

While I was trying to compile Emacs [to test a
fix,](/2019/05/23/how-to-fix-the-emacs-mac-port-for-multi-tty-access/) I ran
into a compilation error. This error occurred during [Emacs' dump
phase](http://emacshorrors.com/posts/unexecute.html) and it said:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
unexec: not enough room for load commands for new __DATA segments (increase headerpad_extra in configure.in to at least 1488)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

I followed this error's advice and I kept increasing the value for
`headerpad_extra` in `configure.ac`. But each time I did so, it still kept
throwing the same error, just with different values like `1638`, `13F8`, `1518`,
etc. I eventually doubled the value of `headerpad_extra` to `2000`, but no luck.
I eventually realized that bumping this value would do no good, so I searched
around for a solution.

Initially, I didn't find much. I found old mailing list threads for unrelated
issues, as well as bug reports for package managers. I also guessed that the
error might have something to do with my OS, which is macOS Mojave at the time
of this writing, but adding this to my search terms did not help. Finally, by
sheer luck, I stumbled upon [a Japanese messageboard
thread](http://anago.2ch.sc/test/read.cgi/mac/1328699139/) which had a fix for
this error:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
LDFLAGS = -headerpad_max_install_names ./configure [your options go here]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

I decided to `export` the `LDFLAGS` for the entire compilation process (just to
be safe):

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ export LDFLAGS=-headerpad_max_install_names
$ echo $LDFLAGS
-headerpad_max_install_names
$ ./configure --with-modules --enable-mac-app --prefix=/Applications/Emacs.app/Contents/Resources && make install
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

And lo and behold, it worked. No more strange dumping errors. I figured that
since this error was difficult to solve and the only fix I found was on an
obscure Japanese messageboard, it might be a good idea to document this problem
in a better format and in English.

Out of curiosity as to why this works, here's an excerpt from [`man ld`:](https://www.unix.com/man-page/OSX/1/ld/)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     -headerpad_max_install_names
                 Automatically adds space for future expansion of load commands such that all paths could expand to MAXPATHLEN.
                 Only useful if intend to run install_name_tool to alter the load commands later.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This still doesn't explain it completely and I'm not 100% sure as to what was
wrong here and why this solution worked, but it did so I'm not complaining.
