# Examples with restic

`restic backup -v --password-file /path/to/password.txt --repository-file /path/to/repo.txt --exclude-file /path/to/excluded.txt`
## Autorestic

https://autorestic.vercel.app/
## The autorestic configuration file

The core of autorestic is its [yaml](https://linuxconfig.org/introduction-to-yaml-with-examples) configuration file. In this file, we set the sources and destinations of our backups, the restic options we want to use, and many other things. For obvious reasons, we can’t cover all possible options here, so we will see only the essential stuff.

Autorestic looks for the configuration file in the following positions, in order of priority:

1. .autorestic.yml
2. ~/.autorestic.yml
3. ~/.config/autorestic/.autorestic.yml

Inside the configuration file, we define the sources of our backups in the “location” section, and their destinations in the “backends” section. Let’s start with a basic example. Suppose we want to backup the content of the `/data` directory to the `/mnt/restic_repo` local repository. Here is how we would populate the autorestic configuration file, which we will save as `.autorestic.yml` in our current working directory:

```yaml
version: 2
locations:
  data:
    from: 
      - /data
    to:
      - localrepo
backends:
  localrepo:
    type: local
    path: /mnt/restic_repo
    key: repositorypassword
```

Each location is identified by a lowercase name; in this case we used “data” for the one we created. With the `from` key, we define one or multiple directories we want to backup: the paths can be specified as absolute or relative to the path of the configuration file. With the `to` key, instead, we provide a single or multiple targets or backends for the location, again, referencing them by name. We define each destination, or “backend”, in the “backends” section.

In this example, we defined just one backend, which we called “localrepo” (just like locations, backends names must be defined in lowercase form). By setting the value of the `type` key to “local”, we specified the backend is a restic repository which exists on the local filesystem (as we know, restic supports a lot of storage platforms; among the others: Backblaze, S3 and SFTP). With the `path` key, we specified the path of the restic repository, and, finally, with `key` we reported the repository password (putting a cleartext password in a configuration file can be dangerous, we will see some alternatives to this behavior later).
## Launching a backup

Now, if it is the first time we run the backup, we can use the autorestic `check` command, to be sure everyhing is ready. The command will automatically initialize the defined backends for us, therefore there is no need to create the restic repositories beforehand:

```sh
sudo autorestic check
```

To start the backup we can now use the `backup` command:

```sh
sudo autorestic backup -va
```

Above, we invoked autorestic with the `-v` option, to make it run in verbose mode, and with `-a` to instruct it to perform the backup of all existing locations. In case we defined multiple locations, and we want to backup only a subset of them, we can pass the comma-separated list of their names as argument to the `-l` option instead, e.g:

```sh
autorestic backup -v -l data
```

## Specifying restic options

In the autorestic configuration file, we can specify the options we want to pass to restic, for each location, or globally. Options restic should be invoked with, are provided as value of the `options` key. We can specify the options we want to pass for the backup and forget commands, or for both of them, using the `backup`, `forget`, and `all` keys, respectively. Here is an example. Suppose we want to exclude all files with the “.txt” extension when performing a backup of the “data” location. We would write:

```json
version: 2
locations:
  data:
    from:
      - /data
    to: 
      - localrepo
    options:
      backup:
        exclude:
          - '*.txt'

backends:
  localrepo:
    type: local
    path: /mnt/restic_repo
    key: repositorypassword
```

The restic “forget” command, is used to remove old snapshots from a repository, according to the policy we specify with the `--keep` options. To provide those options, we specify them under the `forget` key. To always keep all the most recent 10 snapshots and delete the older ones, for example, we would use `--keep-last=10`:

```json
version: 2
locations:
  data:
    from:
      - /data
    to: 
      - localrepo
    options:
      backup:
        exclude:
          - '*.txt'
      forget:
        keep-last: 10

backends:
  localrepo:
    type: local
    path: /mnt/restic_repo
    key: repositorypassword
```

The “forget” options are applied when we invoke autorestic with the `forget` command:

```sh
autorestic forget -va
```

We can configure autorestic so that the “forget” command is automatically executed after each backup, by using the `forget` key directly in the location definition. The value of this key can be set to `true` if we just want the snapshots to be forgotten, or to `prune` if we also want to remove data related to the forgotten snapshots from the repository. Here is an example:

```sh
version: 2
locations:
  data:
    from:
      - /data
    to: 
      - localrepo
    forget: prune
    options:
      backup:
        exclude:
          - '*.txt'
      forget:
        keep-last: 10

backends:
  localrepo:
    type: local
    path: /mnt/restic_repo
    key: repositorypassword
```

The options we set in the configuration file until now, are valid only for the “data” location. If we have multiple locations, and we want to apply certain options to all of them, we can populate the **global** section, e.g:

```sh
version: 2
global:
  backup:
    exclude:
      - '*.txt'
  forget:
    keep-last: 10

locations:
  data:
    from:
      - /data
    to: 
      - localrepo

backends:
  localrepo:
    type: local
    path: /mnt/restic_repo
    key: repositorypassword
```

Location-specific options overiddes those set in the global section. This strategy is useful if we want to just specify exceptions.
## Alternative ways to provide a backend password

In the previous examples, we directly specified the password for the “localrepo” backend in the configuration file. As an alternative, we can set the password as the value of an environment variable we can define when launching autorestic, or in a **dedicated file** called `.autorestic.env`, which must be located in the same directory as the configuration file. In order for the variable to be automatically associated to a specific backend, we define it using the following syntax:

```
AUTORESTIC_<BACKEND-NAME>_<VARIABLE_NAME>
```

So, to assign the password for the “localrepo” backend, we would write:

```
AUTORESTIC_LOCALREPO_RESTIC_PASSWORD="repositorypassword"
```

We can use this strategy also to provide additional information. When using a remote Backblaze backend, for example, we can provide both the account id and account key. Suppose we defined a backup called “backblaze”, we would write:

```
AUTORESTIC_BACKBLAZE_B2_ACCOUNT_ID="123"
AUTORESTIC_BACKBLAZE_B2_ACCOUNT_KEY="456"
```

Another alternative consists into using the restic native `--password-command` option. With this option we can provide a command which must return the password to stdout. With this strategy, for example, we can read the password from an external file, as we did below:

```sh
version: 2
locations:
  data:
    from:
      - /data
    to: 
      - localrepo
    options:
      all:
        password-command: cat /root/.password
      backup:
        exclude:
          - '*.txt'
      forget:
        keep-last: 10

backends:
  localrepo:
    type: local
    path: /mnt/restic_repo
```

We used the option in the “all” section, so that it is always passed, no matter if we run the backup or the forget command.
## Autorestic hooks

Autorestic allows us to execute custom commands in specific moments, using the following hooks:

- prevalidate
- before
- after
- failure
- success

The `prevalidate` hook is run before everything else, even before various checks are executed (autorestic checks if source directories associated to a location exist, for example); the `before` hook, instead, is executed after the checks, but before the backup is launched. The `after` hook is always executed after the backup is finished, independently of the exit status (it is instead skipped if the backup can’t be started at all, due to errors in the “prevalidate” or “before” hooks). Finally, commands associated with the `failure` and `success` hooks, are invoked after the backup is executed successfully, or after it fails, respectively.

In the example below, we send a push notification using the [ntfy.sh service](https://linuxconfig.org/how-to-install-and-self-host-an-ntfy-server-on-linux) if the backup is successful:

```json
version: 2
locations:
  data:
    from:
      - /data
    to: 
      - localrepo
    options:
      all:
        password-command: cat /root/.password
      backup:
        exclude:
          - '*.txt'
      forget:
        keep-last: 10
    hooks:
      success:
        - curl -d "backup executed succesfully!" ntfy.sh/lxcfg

backends:
  localrepo:
    type: local
    path: /mnt/restic_repo
```

During a hook, we can access additional information stored in some predefined variables. Here are some examples:

- AUTORESTIC_SNAPSHOT_ID
- AUTORESTIC_PARENT_SNAPSHOT_ID
- AUTORESTIC_FILES_ADDED
- AUTORESTIC_FILES_CHANGED
- AUTORESTIC_FILES_UNMODIFIED
- AUTORESTIC_LOCATION

https://creativeprojects.github.io/resticprofile/
https://jacobcolvin.com/posts/2023/01/backups-for-k8s-and-beyond/