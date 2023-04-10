---
title: "How to restore tmux lost session"
date: 2023-04-10T08:06:25+06:00
description: How to restore tmux lost session if you have lost your tmux session due to system restart, terminal crash, or any other reason.
# menu:
#   sidebar:
#     name: How to restore tmux lost session
#     identifier: rich-content
#     parent: sub-category
#     weight: 10
# hero: images/forest.jpg
tags: ["tmux", "terminal"]
categories: ["Basic"]
---

Tmux is a terminal multiplexer that allows users to create multiple sessions, panes, and windows within a single terminal window. It is an excellent tool for developers and system administrators who often work with multiple terminal sessions at once. However, sometimes users may face a situation where their tmux session gets lost due to system restart, terminal crash, or any other reason. In this blog post, we will discuss how to restore a lost tmux session.

There are various plugins available for tmux, including tmux-resurrect and tmux-continuum. These plugins allow users to save and restore their tmux sessions, making it easy to recover from a lost session. However, sometimes these plugins may not work correctly, and users may still lose their tmux session. In such cases, users can follow the steps mentioned below to restore their lost tmux session.

## Step 1: Check for session files

The first step is to check if the session files created by tmux-resurrect are present on the system. Tmux-resurrect stores session status files under the ~/.tmux/resurrect directory. Users can check if this directory exists on their system by using the ls command. Usually the files are in the format of `tmux_resurrect_<YYYYMMDD>T<HHMMSS>` format.

```sh
$ ls ~/.tmux/resurrect
```

If the directory exists, users can check if there are any session files present in it. If there are any session files present, it means that tmux-resurrect has saved the session before it was lost.

## Step 2: Restore the session

The next step is to restore the session using the session file saved by tmux-resurrect. In this folder there will be a symlink called `last` that points to the last saved session file. Users can use this symlink to restore the session.

```sh
$ ls -l ~/.tmux/resurrect/last
```

This command will show you the last symlink is pointing to which session file. Users can delete this symlink and create a new symlink to the session file they want to restore.

```sh
$ rm ~/.tmux/resurrect/last
$ ln -s ~/.tmux/resurrect/tmux_resurrect_<timestamp>.txt ~/.tmux/resurrect/last
```

## Step 3: Restore the tmux sever

After repointing the symlink, you will need to kill the tmux server and restart it. This will allow tmux to restore the session from the session file.

```sh
$ tmux detach
$ tmux kill-server
$ tmux
```

```sh
$ tmux list-sessions
```

This command will list all the active tmux sessions. Users should see their restored session listed here.

### Conclusion

In this blog post, we have discussed how to restore a lost tmux session. Users can use the tmux-resurrect and tmux-continuum plugins to save and restore their tmux sessions. However, if these plugins do not work correctly, users can still restore their lost session by following the steps mentioned above. It is essential to keep a backup of the tmux session files to avoid losing any unsaved work.
