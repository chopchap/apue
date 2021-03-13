# tool tips
---
# screen(1)
- screen - screen manager with VT100/ANSI terminal emulation
- `sudo pkgin -y install screen` cmd to download it
- `screen` to enter program screen, `exit` to exit
- `ctrl+a c` to create a second window
- `ctrl+a a` to switch back and forth
- `ctrl+a [0..9]` to go to the numbered window if you've opened multiple windows
- `ctrl+a "` to list windows and keys `j&k` to maneuver as if in vim
- `ctrl+a A` to enter a new title for a window
- besides, all you work will remain saved as long as you're in screen even though network interruption. (once you re-log in, use `screen -r` to reattach a session)
- `ctrl+a |` to vertically split the current window into regions
- `ctrl+a S` to horizontally split ...
- `ctrl+a X` to get back your space
- `ctrl+a tab` to move back and forth bwtween regions
