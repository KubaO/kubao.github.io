---
layout: default
title: Prehistoric Record
parent: Software
permalink: /software/prehistoric
---
{% include switcher.html %}

# Prehistoric Record
{: .no_toc }

It's not retro. It's not vintage. It's just old. Outdated. Outmoded. But in case it were helpful: here it is.
{: .fs-6 .fw-300 }

{% include toc.html %}

---
## Migrating Zimbra 6 to Zimbra 8
*Nov, 2012*

I have a small Zimbra system that needed to be migrated from 6.0 to 8.0. Generally:

1. You must update to 6.0.16 on the source system. The account export functionality is broken in versions prior to 6.0.15 and will fail on some accounts.

2. Start with a fresh 8.0 install. On a small system (dozens of users), that seems like the simplest option.

3. Keep the 6.0 system running.

Use the migration wizard in 8.0's admin console (Tools & Migration -> Account Migration) to move over account records only.

1. Set "type of mail server" to Zimbra Collaboration Suite.
2. Set "would you like to import mail" to No.
3. Import from another Zimbra LDAP directory.
4. Set the LDAP Search Base to "dc=foo,dc=com" where foo.com would be your domain.

This saves work on manually moving accounts between systems.

To move account contents and filter settings between servers, you can do the following on the source server:

```bash
su - zimbra
ACCT=user@foo.com      # account to move
DEST=root@destination  # ssh account on the destination server
zmmailbox -z -m $ACCT getRestURL '//?fmt=tgz' | ssh $DEST "su - zimbra -c 'cat > /tmp/acct'"
ssh $DEST su - zimbra -c 'zmmailbox -z -m $ACCT postRestURL "//?fmt=tar&resolve=reset" /tmp/acct'
zmmailbox -z -m $ACCT gfrl | awk '{print "afrl " $0}' | ssh $DEST "su - zimbra -c 'zmmailbox -z -m $ACCT'"
```

The size of a post request is by default limited to 1GB. You may want to increase it like so:

```bash
zmlocalconfig -e rest_request_max_upload_size=$((1024*1024*1024*4))
```

That's about it.

---
## How to Check for Presence of Trojan-Downloader:OSX/Flashback.I
*Apr, 2011*

If you run OS X and want to check for the presence of Trojan-Downloader:OSX/Flashback.I malware, do the following:

1. Start up the terminal (from Finder, ⌘-Shift-A, click on Utilities, then on Terminal). 
2. Open a new window (⌘-N).
3. Copy-paste the following, followed by a return:

```bash
defaults read /Applications/Safari.app/Contents/Info LSEnvironment || defaults read ~/.MacOSX/environment DYLD_INSERT_LIBRARIES || echo "You're OK-ish"
```

If you get a reply that ends with `You're OK-ish`, then it's more likely that you don't have that trojan, than the opposite.

---
## On Batch-Renaming Files on Unix
*Sept, 2011*

The bash shell has a built-in POSIX regular expression matching facility. It also lets you extract parenthesized matching subexpressions within the regexp. Here's how to use it to rename a bunch of JPGs to jpegs:

```bash
for f in *.JPG; do [[ $f =~ ([^.]+)\.JPG ]] && mv $f ${BASH_REMATCH[1]}.jpeg; done
```

The `[[ .. ]]` test ensures that the rename is only executed when the file name matches the regexp. The `BASH_REMATCH` array is populated while the regexp match operator `=~` does its job. The regular expression within the `[[ .. ]]` cannot be quoted, or else it won't work.

---
## When X Forwarding is Missing on RHEL/CentOS
*Oct, 2011*

I've done a recent sever-style installation of RHEL 6, and tried to use system-config tools via `ssh -X`. It silently failed. Turns out you need to `yum install xorg-x11-xauth`. Easy peasy if you figure it out, that is :)

---
## PDF to PS to PDF Roundtrips on OS X
*circa 2009*

Apple's OS X generates anti-distillation blurbs in the PostScript files generated from "encrypted" PDFs. Remember prohibition, anyone?

The "encrypted", or locked down, rather, PDFs happen to be mostly everything these days. Forms that are meant to be fillable, bank account statements where you want to mark things up to reconcile accounts, etc. My most recent run-in with this stupidity was Anthem's and CompanionLife's insurance forms. I actually wish we didn't have to fill out, um, modify those, right? And surely it's every insurance companies' dream to get the forms back with my dreadful handwriting on them...

So, the pdfs are marked as protected from modification. OS X's otherwise excellent Preview doesn't ignore such marks when you print to PostScript. Thus, the resulting postscript files throw an error when you try to distill them back into pdf, say using `ps2pdf14`.

Upon inspection of the postscript files, you can see the eexec blurb, which can be decoded using ghostscript's decode.ps. The only useful part of the blurb is `cg_md begin`.

Thus, if you want to clean up your postscript files printed from "protected" PDFs, you need to replace stuff between `mark currentfile eexec` and `cleartomark` with `cg_md begin`. This can be done using this handy dandy utility:

```python
#! /usr/bin/env python3
# copy a postscript file from stdin to stdout, removing
# Apple's ps-to-pdf "protection"
import sys;
inside = False
for line in sys.stdin:
    if not inside:
        if line.startswith("mark currentfile eexec"):
            inside = True
        else:
            print(line, file=sys.stdout, end="")
    else:
        if line.startswith("cleartomark"):
            print("cg_md begin", file=sys.stdout)
            inside = False
```

---
## Google Search Defaults To Wrong Country TLD in Opera
*circa 2009*

Opera 10.0 on OS X remembers the non-default country specific google TLD the first time Google redirects it to such a TLD. Having recently visited Australia, my Google search from the toolbar was stuck on google.com.au; clearing cookies and private data didn't help with that.

The culprit is the last line shown belo in `~/Library/Preferences/Opera Preferences:`

```bash
$ cd ~/Library/Preferences/Opera\ Preferences/
$ grep -r 'TLD Default' *
[...]
operaprefs.ini:Keyboard Configuration={Resources}defaults/standard_keyboard.ini
operaprefs.ini:Mouse Configuration={Resources}defaults/standard_mouse.ini
operaprefs.ini:Show Default Browser Dialog=0
operaprefs.ini:Google TLD Default=.google.com.au
```

All it takes to restore the default google TLD (the US one, in my case) is to quit Opera and remove the highlighted line in operaprefs.ini. Due to this browser's cross-platform nature, it's likely that same approach will work on Windows and Linux. Hopefully it will save some traveller a bit of head scratching.

The `~/` refers to your home folder, it a Unix equivalent of Windows's `%USERPROFILE%` environment variable.