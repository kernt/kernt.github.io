# 1. Understand the Basic Syntax

The basic syntax of `rsync` is as follows:

rsync [options] source destination

The `source` can be a file or directory on the local machine or a remote machine, and the `destination` can also be local or remote. For example:

- Local to local:

```sh
rsync -a /source/directory /destination/directory
```

- Local to remote:

`rsync -a /source/directory user@remote:/destination/directory`

- Remote to local:

`rsync -a user@remote:/source/directory /destination/directory`

Knowing this basic structure is key before adding more advanced options.

# 2. Directories Are Not Copied Recursively by Default

By default, `rsync` does **not** recursively copy directories unless you explicitly specify the `-r` (recursive) or `-a` (archive) option. Without these options, `rsync` will only copy individual files at the top level of the specified directory:

`rsync /source/ /destination/`

This command **will not** copy subdirectories.

Let’s have an example. Let’s create two directories, `source` and `destination`, and then create files under the `source`.

Let’s execute the command above ( `rsync source/ destination/` )

The command didn’t copy the files from the `source` to the `destination`

💎 To include all files and subdirectories, always use the `-r` or `-a` option:

`rsync -r /source/ /destination/`

Using `-a` is generally preferred because it includes `-r` along with additional features like preserving permissions, timestamps, and symbolic links.

Here’s an example: We will change the permission of the files in the `source` directory and then `rsync` them to the `destination` with the "`-r`" option and let's see what will happen

After the “`rsync -r`", the files' permissions changed in the `destination` directory.

However “`rsync -a`" saved the same permissions

> **_lways use_** `**_-a_**` **_if you want to maintain the file structure and metadata:_**

# 3. Use the Trailing Slash Wisely

One of the most common pitfalls when using `rsync` is misunderstanding the behavior of the trailing slash (`/`) on the source path:

💎 **Without a trailing slash**: Copies the source directory _and_ its contents into the destination directory.

`rsync -a /source /destination`

Result: `/destination/source`

**With a trailing slash**: Copies only the contents of the source directory into the destination.

`rsync -a /source/ /destination`

Result: `/destination/*contents_of_source*/`

> **_Always double-check whether you need the trailing slash to avoid unexpected directory nesting._**

# 4. Use “-`v”` for Verbose Output

Adding the `-v` (verbose) option to your `rsync` command will provide detailed information about the files being processed. This is helpful for monitoring progress and diagnosing issues:

`rsync -av /source /destination`

For even more detailed output, you can use `-vv`.

# 5. Use “-`-dry-run"` Before Running

`rsync` can make significant changes to your files, so it’s always a good idea to use the `--dry-run` option to simulate the command before executing it. This will display what would be copied, deleted, or updated without making any changes:

`rsync -a --dry-run /source /destination`

> **_This is particularly useful when working with complex commands or critical data transfers._**

# 6. Optimize Transfers with “-`-progress"`

When transferring large files or directories, it’s useful to see the progress of the operation. The `--progress` option shows the file transfer progress, including the speed and estimated time remaining:

`rsync -a --progress /source /destination`

# Use “-`z”` for Compression (When Remote Transfers Are Involved)

When transferring files to or from a remote machine, use the `-z` (compress) option to reduce the data size during transfer. This is particularly helpful for slow network connections:

`rsync -az /source user@remote:/destination`

> **_⚠️ Compression can significantly speed up the transfer of large files such as logs, archives, or backups._**

# 8. Use “`--delete"` Cautiously

The `--delete` option **removes files in the destination that no longer exist in the source**. This is useful for maintaining an exact mirror of the source, but it should be used with caution to avoid accidental data loss.

For example:

`rsync -a --delete /source/ /destination/`

> **_To avoid mistakes, always combine it with_** `**_--dry-run_**` **_first to preview the changes:_**

`rsync -a --delete --dry-run /source/ /destination/`

# 9. Exclude Files or Directories with “`--exclude"`

If you want to skip certain files or directories during the transfer, use the `--exclude` option. This is especially useful when syncing large directories with specific content you don’t need:

`rsync -a --exclude="*.tmp" --exclude="backup/" /source/ /destination/`

For more complex exclusion rules, you can use an exclude file:

`rsync -a --exclude-from='exclude.txt' /source/ /destination/`

The `exclude.txt` file should list patterns of files or directories to skip.

# 10. Use SSH for Secure Transfers

When transferring files between remote machines, always use `rsync` over SSH for secure communication. This is done by adding the `-e ssh` option:

`rsync -a -e ssh /source/ user@remote:/destination`

Alternatively, you can save time by using the shorthand:

`rsync -a user@remote:/source /destination`

# 11. Limit Bandwidth with “-`-bwlimit"`

When transferring files over a shared or slow network, you can limit the bandwidth usage with the `--bwlimit` option. This ensures `rsync` doesn’t consume all available network resources:

`rsync -a --bwlimit=1000 /source user@remote:/destination`

> **_⚠️ The value is in kilobytes per second (e.g.,_** `**_1000_**` **_= 1 MB/s)._**

# 12. Use “`--partial”` to Resume Interrupted Transfers

If a transfer is interrupted, the `--partial` option allows you to resume the transfer without re-transmitting already completed portions of the file. This is especially useful for large file transfers:

`rsync -a --partial /source user@remote:/destination`

> **_⚠️ You may also combine it with_** `**_--progress_**` **_for better visibility._**

# 13. Log Your Transfers

For long-running or automated tasks, it’s helpful to log the output of `rsync`:

`rsync -a --log-file=/path/to/logfile.log /source /destination`

This allows you to review the results or troubleshoot issues later.

# 14. Automate with Cron for Regular Backups

To automate regular backups, you can schedule `rsync` commands using `cron`. For example, to run a backup daily at midnight:

1. Open the crontab editor:

`crontab -e`

2. Add the following line:

`0 0 * * * rsync -a /source/ /backup/ > /path/to/backup.log 2>&1`