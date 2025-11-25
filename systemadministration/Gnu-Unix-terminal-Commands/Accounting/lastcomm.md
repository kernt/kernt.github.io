
lastcomm -  Informationen zu zuvor ausgeführten Befehlen aufzeigen.

- command name of the process
- flags, as recorded by the system accounting routines:
    - **S** command executed by super-user
    - **F** command executed after a fork but without a following exec
    - **C** command run in PDP-11 compatibility mode (VAX only)
    - **D** command terminated with the generation of a core file
    - **X** command was terminated with the signal SIGTERM
- the name of the user who ran the process
- time the process exited

**List Last Executed Commands of User**

`lastcomm tecmint`

**Search Logs for Commands**

`lastcomm ls`

**tty**

`lastcomm a.out tty0`

**tty strict**

`lastcomm --strict-match a.out root tty0`

