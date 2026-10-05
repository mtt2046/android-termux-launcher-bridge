# A Desktop Launcher Bridge Between the Termux Container and Android Apps

## 1. The problem: the container is out of reach
Running a batch of Linux services inside Termux (file server, mail, chat, tunnel, ...) leaves
ordinary users with a real gap: those services live inside the Termux "container". How to start
or stop them conveniently? Typing commands on a phone is hard and unnatural for most people.

## 2. The bridge: a desktop dropdown menu
The fix is Termux:Widget, Termux's official widget app. Long-press the home screen -> Widgets ->
Termux:Widget -> drag it to the desktop, and you "fish out" a list icon. That list reads Termux's
`~/.shortcuts/` directory: each `.sh` script becomes one item in the list. Tapping an item runs a
script that performs a preset action inside the container.

## 3. The point: saying goodbye to the command line (the real breakthrough)
Write the commands into scripts in advance and tuck them into the dropdown. What the user does is
tap once on the desktop. The barrier is flattened - from "can't use it" to "can use it". We call
it a bridge across the container gap.

## 4. A universal launcher: not just inside the container
We first treated it only as an on/off switch for in-container services. Then we realized: each
list item is a script, and a script running in Termux can call Android's `am start` command - so it
can also launch ordinary outside-container apps (WeChat, browser, gallery, ...). That upgrades it
from a "container switch" to a "universal launcher": one desktop icon that governs both in-container
services and outside-container apps.

## 5. How to build it (overview)
1) Install: Termux + Termux:Widget, both from F-Droid (the Play Store versions are discontinued).
2) Make the directory: `mkdir -p ~/.shortcuts/`; each `.sh` script = one list item.
3) Hard rules for the scripts (lessons learned):
   - `chmod +x` to make it executable;
   - use an absolute-path shebang `#!/data/data/com.termux/files/usr/bin/sh` (not `#!/bin/sh` or `#!/usr/bin/env`);
   - export HOME/PREFIX/PATH yourself at the top of the script;
   - detach long-running processes with `setsid nohup ... & < /dev/null`, or they die right after the tap.
4) Put it on the desktop: long-press -> Widgets -> Termux:Widget -> drag out.
Minimal example (just starts sshd):
```sh
#!/data/data/com.termux/files/usr/bin/sh
export HOME=/data/data/com.termux/files/home
export PREFIX=/data/data/com.termux/files/usr
export PATH=$PREFIX/bin:/system/bin:/system/xbin:$PATH
sshd
```
Launching an outside-container app is also just one line:
```sh
am start -p com.tencent.mm        # open WeChat; swap the package name. On ROMs where -p fails, use: am start -n <pkg>/<mainActivity>
```
Drop these lean scripts into `~/.shortcuts/` one by one, and the menu grows - governing both
in-container services and outside-container apps with a single tap.

## 6. Multiple launchers? Currently limited
Termux:Widget reads only the single `~/.shortcuts/` directory; every widget instance you drag out
points to it. So for now it's "one launcher"; when the list gets long, group items with
subdirectories (e.g. `server/`, `apps/`) rather than multiple icons.

## 7. Summary
Use a desktop widget as a bridge to fold in-container and out-container operations into one dropdown
menu, done with a single tap. For phone users who'd rather not touch the command line, this is a
real barrier-breaking win.

(This is the framework version: the idea and the hard rules are all here; write your own full scripts following the rules above - no fixed set to copy.)
