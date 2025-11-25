# Linux expect Command Syntax


**Password Send**

```sh
#!/usr/bin/expect
set timeout 20
spawn sudo passwd ubuntu
expect "Enter new UNIX password:" {send "ubuntu\\r"}
expect "Retype new UNIX password:" {send "ubuntu\\r"}
interact' > change_ubuntu_password
   chmod +x change_ubuntu_password
```

The Except program uses the following keywords to interact with other programs

|Command|Function|
|---|---|
|**`spawn`**|Creates a new process.|
|**`send`**|Sends a reply to the program.|
|**`expect`**|Waits for output.|
|**`interact`**|Enables interacting with the program.|

expect Options

|Command|Description|
|---|---|
|**`-c`**|Specifies the command to execute before the script.|
|**`-d`**|Provides a brief diagnostic output.|
|**`-D`**|Interactive debugger.|
|**`-f`**|Specifies a file to read from.|
|**`-i`**|Prompts commands interactively.|
|**`-b`**|Reads file line by line (buffer).|
|**`-v`**|Print version.|
### Basic Expect Use

Below is a basic example that explains how the **`expect`** command functions:

1. Open a [text editor](https://phoenixnap.com/kb/best-linux-text-editors-for-coding) and name the file _interactive_script.sh_. If you use Vim, run:

```sh
vim interactive_script.sh
```

2. Add the following code to the file:

```sh
#!/bin/bash

echo "Hello, who is this?"
read $REPLY
echo "What's your favorite color?"
read $REPLY
echo "How many cats do you have?"
read $REPLY
```

![interactive_script.sh contents](https://phoenixnap.com/kb/wp-content/uploads/2022/06/interactive_script.sh-contents.png)

The code is a basic script with the [read command](https://phoenixnap.com/kb/bash-read) that expects user interaction when run.

1. Save the file and close Vim:

```sh
:wq
```

2. Change the script to executable:

```sh
chmod +x interactive_script.sh
```

3. Create a new file to store the Expect script with:

```sh
vim expect_response.exp
```

The _.exp_ extension is not mandatory, though it helps differentiate Expect scripts from other files.

4. Add the following code to the script:

```sh
#!/usr/bin/expect

spawn ./interactive_script.sh
expect "Hello, who is this?\r"
send -- "phoenixNAP\r"
expect "What's your favorite color?\r"
send -- "Blue\r"
expect "How many cats do you have?\r"
send -- "1\r"
expect eof
```

![expect_response.exp contents](https://phoenixnap.com/kb/wp-content/uploads/2022/06/expect_response.exp-contents.png)

The script consists of the following lines:

- **`spawn`** creates a new process running the _interactive_script.sh_ file.
- **`expect`** writes the expected program message and waits for the output. The final line ends the program.
- **`send`** contains the replies to the program after each expected message.

5. Save the file and close:

```sh
:wq
```

6. Make the script executable:

```sh
chmod +x expect_response.exp
```

7. Run the script with:

```sh
expect expect_response.exp
```

Or alternatively:

```sh
./expect_response.exp
```

![expect expect_response.exp terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/06/expect-expect_response.exp-terminal-output.png)

The _expect_response.exp_ script spawns the program process and sends automatic replies.

### Expect with Variables

Use the [set command](https://phoenixnap.com/kb/linux-set) to store variables and pass values in Expect scripts. For example, to [hardcode](https://phoenixnap.com/glossary/hardcoded) a variable, use:

```sh
set MYVAR 5
```

For user input arguments use:

```sh
set MYVAR1 [lindex $argv 0]
set MYVAR2 [lindex $argv 1]
```

In both cases, reference the variable in the script with **`$<variable name>`**.

The following example demonstrates using both variables in the Expect script from the previous example (_expect_response.exp_):

```sh
#!/usr/bin/expect
set NAME "phoenixNAP"
set COLOR "Blue"
set NUMBER [lindex $argv 0]
spawn ./interactive_script.sh
expect "Hello, who is this?\r"
send -- "$NAME\r"
expect "What's your favorite color?\r"
send -- "$COLOR\r"
expect "How many cats do you have?\r"
send -- "$NUMBER\r"
expect eof
```

![expect script with variables](https://phoenixnap.com/kb/wp-content/uploads/2022/06/expect-script-with-variables.png)

Passing a number specifies the **`$NUMBER`** variable:

```sh
./expect_response.exp 22
```

![expect_response.exp variables terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/06/expect_response.exp-variables-terminal-output.png)

The other two variables (**`$NAME`** and **`$COLOR`**) are hardcoded.

### Expect with Commands

Use the **`expect`** command to automate responses to other programs and commands.

For example, the [passwd command](https://phoenixnap.com/kb/passwd-command-in-linux) prompts the user to input the password twice. While the process is simple for one user, difficulties arise when adding a default password for hundreds of new users as a system administrator.

Expect easily automates the responses to other commands.

8. The following script provides an example of using the **`expect`** command with the **`passwd`** command:

```sh
#!/usr/bin/expect
set USER [lindex $argv 0]
set PASS [lindex $argv 1]
set timeout 1
spawn passwd $USER
expect -exact "Enter new UNIX password: "
send -- "$PASS\r"
expect -exact "Retype new UNIX password: "
send -- "$PASS\r"
expect eof
```

![expect password user input script](https://phoenixnap.com/kb/wp-content/uploads/2022/06/expect-password-user-input-script.png)

The code takes two passed arguments as the username and password to provide for the **`passwd`** command.

9. Run the script with:

```sh
sudo expect password.exp <username> <password>
```

![sudo expect passwd.exp terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/06/sudo-expect-passwd.exp-terminal-output.png)

The script automatically sets the password for the provided username.

10. To automate the task for multiple users, use a **`while`** loop. The syntax for the TCL while loop is different from the Bash while loop:

```sh
#!/usr/bin/expect
set PASS "Welcome@123"
set i 1
set timeout 1
while {$i<11} {
        spawn passwd "user$i"
        expect -exact "Enter new UNIX password: "
        send -- "$PASS\r"
        expect -exact "Retype new UNIX password: "
        send -- "$PASS\r"
        expect eof
                incr i
}
```

![expect password change multiuser](https://phoenixnap.com/kb/wp-content/uploads/2022/06/expect-password-change-multiuser.png)

The script assumes all users have the username **`user1`**, **`user2`**, etc., up to **`user10`**. The password is hardcoded as **`Welcome@123`**. After each user, the code increments the **`i`** value.

11. Run the script with:

![sudo expect passwd.exp multiuser terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/06/sudo-expect-passwd.exp-multiuser-terminal-output.png)

For the code to work, the example assumes the users already exist on the system.

**Note:** Exposing the script in any way makes the default password vulnerable. After this step, all users should change their password to a [stronger password](https://phoenixnap.com/blog/strong-great-password-ideas) and opt-in for a [password manager](https://phoenixnap.com/glossary/what-is-a-password-manager).
### Autoexpect

Instead of writing Expect scripts from scratch, the Autoexpect program helps generate scripts interactively.

To demonstrate how Autoexpect works, do the following:

1. [Run the Bash script](https://phoenixnap.com/kb/run-bash-script) from the first example (_interactive_script.sh_) using Autoexpect:

```sh
autoexpect ./interactive_script.sh
```

![autoexpect script generation terminal output](https://phoenixnap.com/kb/wp-content/uploads/2022/06/autoexpect-script-generation-terminal-output.png)

The output prints a confirmation message and the Expect script name (_script.exp_) to the console.

2. Provide answers to the questions. The replies save to the _script.exp_ file and generate the Expect program code automatically. When complete, the output prints a confirmation.

3. Review the generated script in a text editor to see the code:

```sh
vim script.exp
```

![autoexpect script contents](https://phoenixnap.com/kb/wp-content/uploads/2022/06/autoexpect-script-contents.png)

The interactions are saved to the Expect script for future use.

`sshsudologin.expect` Skript

```sh
#!/usr/bin/expect
#Usage sshsudologin.expect <host> <ssh user> <ssh password> <su user> <su password>

set timeout 60
spawn ssh [lindex $argv 1]@[lindex $argv 0]

expect "yes/no" { 
	send "yes\r"
	expect "*?assword" { send "[lindex $argv 2]\r" }
	} "*?assword" { send "[lindex $argv 2]\r" }

expect "# " { send "su - [lindex $argv 3]\r" }
expect ": " { send "[lindex $argv 4]\r" }
expect "# " { send "ls -ltr\r" }
interact
```

### [Important Points about Expect Script](https://www.digitalocean.com/community/tutorials/expect-script-ssh-example-tutorial#important-points-about-expect-script)[](https://www.digitalocean.com/community/tutorials/expect-script-ssh-example-tutorial#important-points-about-expect-script)

1. Notice the first line that specifies that expect script will be used as interpreter.
2. Expect script default timeout is 10 seconds, so I have set the timeout to 60 seconds to avoid any timeout issues if the login prompt takes time to come.
3. Notice the `expect` command followed by the regular expression and then what should be send as a response. The first Yes/No choice is added to make sure that it doesn’t fail if remote server key is not already imported.
4. The other expect [regular expressions](https://www.digitalocean.com/community/tutorials/regular-expression-in-java-regex-example) varies across the servers, for my server it ends with “#” but for some other servers it might end with “$”, so you may need to edit it accordingly. The only thing to check is that the regular expression for the expect command should matches, so that it will send the corresponding command or password.
5. The last expect command just shows that we can send commands also once we are logged into the server.

### [Extra Tips on expect script](https://www.digitalocean.com/community/tutorials/expect-script-ssh-example-tutorial#extra-tips-on-expect-script)[](https://www.digitalocean.com/community/tutorials/expect-script-ssh-example-tutorial#extra-tips-on-expect-script)

1. The expect script can be reused since all the information is passed as arguments, so it’s best to create alias for each one of them for quick login and save your time in typing all the parameters. For example, `alias journal="/Users/pankaj/scripts/sshloginsudo.expect 'journaldev.com' 'pankaj' 'ssh_pwd' 'su_user' 'su_pwd'"`
2. It’s always best to use single quotes to pass the arguments, since most of the time passwords contains special characters that can cause funny results if they are not quoted.
## Automate SSH Login with expect

In this step, you will learn how to use the `expect` command to automate the SSH login process.

Let's start by creating an `expect` script to automate the SSH login:

```sh
#!/usr/bin/expect -f
set timeout 10 
set host "example.com" 
set user "myuser" 
set password "mypassword" 

spawn ssh $user@$host 
expect "password:" 
send "$password\r" 
expect "$" 
send "echo 'SSH login successful!'\r" 
expect eof
```

## Handle Prompts and Responses in Expect Scripts

In this step, you will learn how to handle different types of prompts and responses in your `expect` scripts.

Let's create a new `expect` script that demonstrates handling various prompts and responses:

```sh
#!/usr/bin/expect -f 
set timeout 10 
set host "example.com" 
set user "myuser" 
set password "mypassword" 
spawn ssh $user@$host 
expect { 
		"password:" { 
		   send "$password\r"
		   expect "$" 
		}
		 "yes/no" { 
           send "yes\r" 
           expect "$" 
        } 
          "$" { 
            send "echo 'SSH login successful!'\r"
            expect "$"
        } 
        timeout { 
            puts "SSH login timed out"
            exit 1 
        } 
} 
send "exit\r" 
expect eof
```

In this script, we use the `expect` command with a block of `expect` statements to handle different types of prompts:

1. If the script encounters the "password:" prompt, it sends the password and waits for the shell prompt.
2. If the script encounters a "yes/no" prompt, it sends "yes" and waits for the shell prompt.
3. If the script encounters the shell prompt (`$`), it executes the `echo` command to verify the SSH login.
4. If the script encounters a timeout, it prints a message and exits with an error code.

Finally, the script sends the `exit` command to close the SSH session and waits for the end of the script (`expect eof`).

Save this script as `handle_prompts.exp` in your `~/project` directory.

Now, let's make the script executable and run it