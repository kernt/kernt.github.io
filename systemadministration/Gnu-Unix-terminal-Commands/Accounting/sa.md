
summarizes the information in the `acct` file into the `savacct` and `usracct` file. It also generates reports about commands, giving the number of invocations, cpu time used, average core usage, etc.

### Print All Linux Commands Executed by Users

The **sa** command is used to print the summary of commands that were executed by users.

```sh
sa
```

### Print Linux User Information

To get the information of an individual user, use the options **-u**.

```sh
sa -u
```

### Print Number of Linux Processes

This command prints the [total number of processes](https://www.tecmint.com/find-processes-by-memory-usage-top-batch-mode/ "Find Top Processes by Memory Usage") and CPU minutes. If you see a continued increase in these numbers, then it’s time to look into the system about what is happening.

`sa -m`

### Print and Sort Usage by Percentage

The command “**sa -c**” displays the highest percentage of users.

`sa -c`


