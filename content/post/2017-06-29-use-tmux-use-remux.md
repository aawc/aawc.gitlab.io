---
title: Use tmux? Use remux!
subtitle: Making tmux easier to use
date: 2017-06-29
tags: ["bash", "code", "tmux"]
---

Let's make it easier to use ```tmux``` by using ```remux``` instead.

<!--more-->

### ```tmux``` is awesome
Do you use ```tmux```? If not, you should really consider using it.
Here's a [simple tutorial]
(https://www.digitalocean.com/community/tutorials/how-to-install-and-use-tmux-on-ubuntu-12-10--2)
that looks short and useful. I should write one of my own here.

### My problem with ```tmux```
Now, my biggest problem with ```tmux``` is that I could never remember the
syntax for starting a new ```tmux``` session, compared to resuming a new
session.

Here's how you start a new session named ```aawc```:
```bash
$ tmux new -s aawc
```
That's not so hard to remember, now is it?

Now here's what you need to do if you want to resume an existing ```tmux```
session named ```aawc```:
```bash
$ tmux new -t aawc
```

You'd notice that it uses a different switch than creating a new session, which I
can understand, but I can never remember which switch to use when.

And then, if you wanted to list all existing sessions, you did:
```bash
$ tmux list-sessions
```

So, depending on what you're trying to do, you either use a switch, or a
different switch, or a full term. Clearly, not the easiest options to remember.

### My solution: ```remux```
Enter ```remux```, a utility that I wrote in ```bash``` to help me do The Right
Thing™ automatically.

```bash
#!/bin/bash

function __remux__
{
  local muxName="${1}"
  local runningMuxs="$(tmux list-sessions 2>/dev/null | cut -f1 -d:)"

  if [ -z "${muxName}" ]; then
    if [ -z "${runningMuxs}" ]; then
      printf "No muxs running.\n"
      return
    fi

    printf "Found the following muxs running:\n${runningMuxs}\n"
    return
  fi

  local intendedMuxName="$(echo ${runningMuxs} | tr ' ' '\n' | grep -i "${muxName}")"
  if [ -z "${intendedMuxName}" ]; then
    printf "Mux '%s' not found.\n" $muxName
    if [ -n "${runningMuxs}" ]; then
      printf "Found the following muxs running:\n${runningMuxs}\n"
    fi
    local startMux="N"
    read -r -p "Want to start a session named '${muxName}'? [y/N]: " startMux
    case "${startMux}" in
      [yY][eE][sS]|[yY])
        __newTmuxSession__ "${muxName}"
        ;;
      *)
        ;;
    esac
  else
    local numberOfMatchingMuxsFound="$(echo ${intendedMuxName} | wc -w)"
    if [ "${numberOfMatchingMuxsFound}" -eq 1 ]; then
      tmux attach -d -t "${intendedMuxName}"
    else
      printf "Found too many muxs:\n%s\n" "${intendedMuxName}"
    fi
  fi
}

function __newTmuxSession__
{
  local muxName="${1}"
  local tmuxinatorSessions="$(tmuxinator list 2>/dev/null | grep ${muxName})"
  if [ -n "${tmuxinatorSessions}" ]; then
    if [ $(wc -l <<< "${tmuxinatorSessions}") -eq "1" ]; then
      tmuxinator "${tmuxinatorSessions}"
    else
      # Untested
      printf "Too many matching projects found:\n${tmuxinatorSessions}\n"
    fi
  else
    # tmuxinator is either not installed or the project isn't available.
    tmux new -s "${muxName}"
  fi
}

__remux__ "$@"
```

### Usage

And now, all I ever need to do is:
```bash
$ remux aawc
```

* Start a new session (after confirmation) if one doesn't already exist:
```bash
$ remux aawc
Mux 'aawc' not found.
Want to start a session named 'aawc'? [y/N]: y
(New session named 'aawc' started.)
```

* Resume a session by entering its name:
```bash
$ remux aawc
(Existing session named 'aawc' is resumed)
```

* Resume a session by entering a sub-string:
```bash
$ remux a
(Existing session named 'aawc' is resumed)
```

* List the currently running sessions:
```bash
$ remux varun
Mux 'varun' not found.
Want to start a session named 'varun'? \[y/N\]: y
(New session named 'varun' started)
$
$ remux
Found the following muxs running:
'aawc'
'varun'
```

So, there you have it folks. ```remux``` away!

### Code
The code for ```remux``` is inside my [dotfiles]
(https://github.com/aawc/dotfiles) repository
[here](https://github.com/aawc/dotfiles/blob/09df0db176f322d3f799b973f3955ef28380a84e/rc/common/functions/remux).

PRs are most welcome.
