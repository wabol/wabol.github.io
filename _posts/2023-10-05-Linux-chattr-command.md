---
title: Linux chattr command 
date: 2023-10-05 11:11:00 -0700
categories: [Linux]
tags: shell
Pin:
---

chattr (Change Attribute) is a command line Linux utility that is used to set/unset certain attributes to a file in Linux system to secure accidental deletion or modification of important files and folders, even though you are logged in as a root user.

chattr parameter and usage

| Parameter             | Usage                                                        |
| --------------------- | ------------------------------------------------------------ |
| +a (Append Only)      | Set the "append-only" attribute. Files with this attribute can only be opened in "append" mode for writing. |
| +i (Immutable)        | Set the "immutable" attribute. Files with this attribute cannot be modified, deleted, or renamed, even by the root user. |
| +c (No-Copy On Write) | Disable the copy-on-write feature for files in a btrfs file system. |
| +u (Undeletable)      | Mark the file as undeletable. Once set, the file cannot be deleted until this attribute is removed. |
| +s (Secure Deletion)  | Enable secure deletion of a file. When a file with this attribute is deleted, its data blocks are overwritten with zeros. |
| +S (sync)             | When a file with the 'S' attribute set is modified, the changes are written synchronously to the disk; this is               equivalent to the 'sync' mount option applied to a subset of the files. |
| +A (atime)            | its time record is not modified.  This avoids a certain amount of disk I/O for laptop systems. |
| +u                    | its contents are saved.  This allows the user to ask for its undeletion. |

**'-' every parameter means remove attribute. and it can be displayed by 'lsattr'**

