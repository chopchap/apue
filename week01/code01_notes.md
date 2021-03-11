# Code01_notes
---
## simple-shell.c
- **fgets:** *get a line from a stream, EOF(ctrl+D) to quit*
- **errno** *will be set if some error occurs, and it'll be a number, so better use strerror in additon to offer clarity*
## simple-ls.c
- **opendir & readdir:** *to list contents in a dir, open it first, then readdir, return types are `DIR *` and `struct dirent *` respectively*
- **closedir**: *don't forget to close*