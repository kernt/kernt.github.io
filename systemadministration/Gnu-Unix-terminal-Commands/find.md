---
tags:
  - system-administration
  - bash
  - gnu-tools
---
```sh
sudo dpkg -l | grep libghc | grep "\-dev" | cut -d " " -f 3 | tr '\n' ' ' | sed -e 's/\-dev/\-prof/g' | xargs sudo apt-get install --yes
```

`find . -name *conf* -exec grep -Hni 'matching_text' {} \; > matching_text.conf.list`

**Änderungen an Dateien seit den Letzten 10Minuten**

`find . -type f -mmin -10`
## Understanding find -exec Command Option

The -exec option in the find command of Linux stands out as a cornerstone for executing commands on files that meet specified search criteria.
### Syntax of find -exec Command

The -exec option is used as part of the find command. The syntax is:

```bash
find [path] [expression] -exec [command] {} \;
```

- Defining the Search Path [path]: This is where the command starts searching. It can be a specific directory or a broader location, depending on the user’s requirements.
- Setting the Search Criteria [expression]: This powerful segment allows users to specify what files to look for. It could be based on file names, types, size, modification dates, and other attributes.
- Executing the Command [command]: Here, users define the action on the found files. This could range from simple operations like deleting or moving files to more complex tasks such as modifying content or changing permissions.
- Placeholder {} for Current File: A critical part of the syntax, {} is replaced by the current file name being processed in each command iteration.
- Terminating the Command Sequence \;: This marks the end of the -exec command, signaling the completion of one set of command executions.

Moving forward, the guide will transition to providing practical examples followed by a series of more advanced applications of the find -exec command option.

## Practical Examples Using the find -exec Command Option

The following sections provide unique and detailed examples demonstrating the versatility of the find command combined with the -exec option in Linux.

**Backing Up Files with find -exec**

To find and create backups of all .jpg files in the /pictures directory:

```bash
find /pictures -type f -name "*.jpg" -exec cp {} {}.backup \;
```

This command locates each .jpg file and creates a backup by copying each file to a new file with the .backup extension.

**Renaming File Extensions Using find -exec**

To change the extension of all .html files to .htm in the /web directory:

```bash
find /web -type f -name "*.html" -exec sh -c 'mv "$0" "${0%.html}.htm"' {} \;
```

This command renames each .html file, replacing the extension with .htm.

**Converting Image Formats with find -exec**

To convert all .png images to .jpg in the /images directory:

```bash
find /images -type f -name "*.png" -exec convert {} {}.jpg \;
```

This uses the convert command (from the ImageMagick suite) to change each .png file to a .jpg file, keeping the original files.

**Compressing Log Files: A find -exec Approach**

To find and compress all .log files older than 7 days in /var/log:

```bash
find /var/log -type f -name "*.log" -mtime +7 -exec gzip {} \;
```

This command selects .log files older than 7 days and compresses them using gzip.

**Removing Empty Directories with find -exec**

To find and remove all empty directories in the /data directory:

```bash
find /data -type d -empty -exec rmdir {} \;
```

This command identifies empty directories within /data and removes them, streamlining the file system.
# Advanced Use Cases for the find -exec Option

This section delves into more complex scenarios, addressing commonly asked questions and challenging tasks that can be efficiently handled using the find command with the -exec option. These examples are tailored for specific, advanced use cases, ensuring the commands are practical, relevant, and functional.
### Syncing Files to Remote Servers: Advanced find -exec Usage

To synchronize all .pdf files from /local/docs to a remote server:

```bash
find /local/docs -type f -name "*.pdf" -exec rsync -avz {} user@remote_server:/remote/docs/ \;
```

This command finds all .pdf files and uses rsync to synchronize them with a specified directory on a remote server, ensuring efficient data transfer and backup.
### Date Stamping File Names: A find -exec Technique

To add a current date stamp to the filenames of all .csv files in /data/reports:

```bash
find /data/reports -type f -name "*.csv" -exec sh -c 'mv "$0" "$(dirname "$0")/$(date +%Y%m%d)-$(basename "$0")"' {} \;
```

This command locates .csv files and renames each by prefixing the current date, enhancing file organization and version control.
### Generating Large File Reports via find -exec

To find files larger than 100MB in /home and email a report:

```bash
find /home -type f -size +100M -exec ls -lh {} \; | mail -s "Large Files Report" admin@example.com
```

This command identifies files over 100MB, lists their details, and sends this information via email, assisting in capacity management and monitoring.
### Automated Image Watermarking with find -exec

To add a watermark to all .jpg images in /images/gallery:

```bash
find /images/gallery -type f -name "*.jpg" -exec composite -dissolve 30% -gravity southeast watermark.png {} {} \;
```

This uses the composite command (part of ImageMagick) to overlay a watermark image on each .jpg file, crucial for copyright protection and branding.
### Directory Creation Based on File Names Using find -exec

To create directories based on the names of .mp4 files in /videos:

```bash
find /videos -type f -name "*.mp4" -exec sh -c 'mkdir -p "/archive/$(basename "{}" .mp4)"' \;
```

This command extracts the base name of each .mp4 file and creates a corresponding directory in /archive, useful for organized storage of related files.
