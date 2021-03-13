# tool tips
---
## ctags
- ctags(1) is a Unix tool to index your code base and create a mapping of where functions are defined.
- once you `ctags *` the source files, then a file named *tags* will be created and we now have the mapping that tells the editor(e.g. vim) that for any given tag it can open specified file and search for the given *regex* to find the definition.
- `ctrl + ]` to jump onto and `ctrl + t` to jump back
- or type `:ta func_name` to be brought to the func_name definition code
- in order to do the same thing including all the C libraries codes, once you've downloaded the source code (e.g. in /usr/src/lib/libc/stdio for all stdio implementaion), you can create a global text file that indexes the entire source tree, and add that to our editor's list of tag files to look things up in (we use exuberant ctags since this version of ctags(1) supports ignoring certain patterns, such as macro definitons that might trip up the regex used in finding the tags). and cmd `exctags -f ~/.tags -R -I __weak_alias /usr/include /usr/src/lib` can handle it.
- for jumping directly to the mannual page by pressing `shift + k`
