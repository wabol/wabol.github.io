---
title: Linux command 
date: 2023-10-04 11:11:00 -0700
categories: [Linux]
tags: shell
Pin:
---

### setfacl

setfacl sets (replaces), modifies, or removes the access control list (ACL) to regular files and directories. It also updates and deletes ACL entries for each file and directory that was specified by path. 

#### Syntax

```shell
setfacl [-bkndRLPvh] [{-m|-x} acl_spec] [{-M|-X} acl_file] file ...

setfacl -Rm u:www:rwx /root # u=user
setfacl -Rm g:user_group:r-x /tmp  # g=group
```

```shell
setfacl --restore=file
```

#### Options

| Parameter        | Usage                                                        |
| ---------------- | ------------------------------------------------------------ |
| -b, --remove-all | Remove all extended ACL entries. The base ACL entries of the owner, group and others are retained. |
| -R, --recursive  | Apply operations to all files and directories recursively. This option cannot be mixed with "--restore". |
| -m, --mask       | Do recalculate the effective rights mask, even if an ACL mask entry was explicitly given. (See the -n option.) |
| -n, --no-mask    | Do not recalculate the effective rights mask. The default behavior of setfacl is to recalculate the ACL mask entry, unless a mask entry was explicitly given. The mask entry is set to the union of all permissions of the owning group, and all named user and group entries. (These are exactly the entries affected by the mask entry). |
| --restore=file   | Restore a permission backup created by "getfacl -R" or similar. All permissions of a complete directory subtree are restored using this mechanism. If the input contains owner comments or group comments, setfacl attempts to restore the owner and owning group. If the input contains flags comments (which define the setuid, setgid, and sticky bits), setfacl sets those three bits accordingly; otherwise, it clears them. This option cannot be mixed with other options except "--test". |
| --test           | Test mode. Instead of changing the ACLs of any files, the resulting ACLs are listed. |

**getfacl use for check the ACL information.** `getfacl file_name` `getfacl /directory_name`
