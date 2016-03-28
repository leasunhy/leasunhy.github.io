---
title: How to Read Relocation Table and Symbol Table From ELF File
date: 2015-11-8 22:52:13
tags:
  - Binutils
  - Linux
---

When studying computer organization and writing an operating system, techniques to find out what entries (and the addresses) are in the symbol table of the ELF file are sometimes useful.

If you've compiled the source file into a object file and want to take a look into its symbol table, simply use a command as follows:
```bash
$ readelf --symbols SOME_ELF_FILE
```

If you are interested in its relocation table, just change that `--symbols` option, i.e.:
```bash
$ readelf --relocs SOME_ELF_FILE
```

