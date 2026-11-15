# Shelving work in Perforce

Shelving stores your pending changes on the server without submitting them. Use it to back up work, hand work to a colleague, or put work aside and switch tasks.

This guide covers the full flow: reconcile offline work, shelve it, then revert your local changes.

## Prerequisites

- A Perforce workspace connected to the server (`p4 info` works).
- Command line (`p4`) or P4V. Both are shown below.

## 1. Reconcile offline work

Offline work means you edited, added, or deleted files without Perforce knowing (for example, Unreal Editor was run while disconnected, or files were made writable by hand). Perforce does not track these changes until you reconcile.

Reconcile first. Shelve only includes files that are opened in a pending changelist.

### Preview

```sh
p4 reconcile -n //...
```

`-n` shows what would happen and changes nothing. Check the list. Make sure generated folders (`Intermediate`, `Saved`, `Binaries`, `DerivedDataCache`) are not in it. If they are, fix your `.p4ignore` first.

### Reconcile into a changelist

Create a numbered changelist and reconcile into it:

```sh
p4 change
```

Note the changelist number, for example `12345`. Then:

```sh
p4 reconcile -c 12345 //...
```

Useful flags:

| Flag | Effect |
|------|--------|
| `-e` | Only open files modified offline for edit |
| `-a` | Only open new files for add |
| `-d` | Only open missing files for delete |

Narrow the path to limit scope, for example `//...` becomes `Content/...`.

### P4V

1. Right-click the workspace folder.
2. Choose **Reconcile Offline Work...**
3. Review the list and choose the target changelist.

## 2. Shelve

Put files already opened in changelist `12345` on the server:

```sh
p4 shelve -c 12345
```

Add a clear description to the changelist so others know what the shelf contains (`p4 change 12345`).

Verify the shelf before you touch your local files:

```sh
p4 describe -S 12345
```

Every file you want to keep must appear in the output.

To update an existing shelf with newer edits, replace it:

```sh
p4 shelve -r -c 12345
```

In P4V: right-click the changelist and choose **Shelve Files...**.

## 3. Revert local changes

Revert local changes after the shelf is verified.

> **Warning:** Revert discards your local edits. The shelf is then the only copy. Do not revert until `p4 describe -S 12345` shows every file.

```sh
p4 revert -c 12345 //...
```

This reverts every file opened in changelist `12345`. Edited files return to the depot version. Files opened for add are no longer tracked. The new file stays on disk.

To keep the files on disk and only close them in Perforce, use `-k`:

```sh
p4 revert -k -c 12345 //...
```

Check that the workspace is clean:

```sh
p4 opened
p4 reconcile -n //...
```

Both must show nothing for the paths you shelved. A leftover file from an added path appears in `reconcile -n`. Delete it by hand if you do not want it.

In P4V: right-click the changelist and choose **Revert Files**.

## 4. Restore the work later

```sh
p4 unshelve -s 12345 -c <target changelist>
```

- `-s` is the source shelf.
- `-c` is the changelist that receives the files. Omit it to use the default changelist.
- If a file is already opened in your workspace, use `-f` to overwrite it. Revert or shelve that file first.

Resolve conflicts with `p4 resolve`. Binary assets (`.uasset`, `.umap`) cannot be merged. Choose one version with `p4 resolve -at` (take theirs, the shelved file) or `p4 resolve -ay` (keep yours).

## 5. Delete the shelf

Delete the shelf when it is no longer needed:

```sh
p4 shelve -d -c 12345
```

## Notes for Unreal projects

- Close Unreal Editor before you revert. The editor can hold files open and rewrite them.
- Shelve code and content together so the project stays consistent after unshelve.
- Do not shelve generated folders. Keep them in `.p4ignore`.
- Unreal Editor's built-in source control can also check out and shelve, but offline reconcile is simpler with P4V or the command line.

## Quick reference

```sh
p4 reconcile -n //...             # preview offline changes
p4 reconcile -c 12345 //...       # open them in changelist 12345
p4 shelve -c 12345                # shelve
p4 describe -S 12345              # verify shelf
p4 revert -c 12345 //...          # revert local changes
p4 unshelve -s 12345 -c 12346     # restore
p4 shelve -d -c 12345             # delete shelf
```
