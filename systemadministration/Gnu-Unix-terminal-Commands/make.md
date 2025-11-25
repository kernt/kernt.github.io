# How does it work?

`make`uses a declarative programming language to determine the different actions to be performed.

These actions are defined in a file called `Makefile`and containing the different instructions used for compilation but also other tasks (installation / uninstallation, cleaning, etc.).

`make`will look for the following file names in the current folder by default: `GNUMakeFile`, `makefile`, `Makefile`. It is of course possible to specify the file to use with the `--file`or argument `-f`.

`make`uses dates to determine what needs to be (re)compiled. If the source file (the dependency) is newer than the binary (the target), then the instructions to build the target will be run.

# The language

Makefiles _therefore_ contain the different instructions written in a specific language. We will see the most important concepts here.

# The targets

They represent the actions to be performed, such as building a binary from source. They are followed by their dependencies (on the same line). Below are the instructions needed to perform our action. These instructions must be indented with a tab. Here is a simple example:

```
build: text_1 text_2  
    cat text_1 text_2 > build  
text_1:  
    echo "Hello" > text_1  
text_2:  
    echo "World" > text_2
```

The target `build`depends on the files `text_1`and `text_2`, these files also happen to be targets without dependencies, they just allow to create text files using a simple command `echo`.

If one of the two files used as dependencies is `build`not present, then `make`will execute the target to create it. If _build_ does not exist, or if its creation date is older than the creation dates of its dependencies (_text_1_ and _text_2_), then the target will be executed.

To launch a target, simply execute `make`with the target as a parameter:

```
make build  
echo "Hello" > text_1  
echo "World" > text_2  
cat text_1 text_2 > build
```

These new files have been successfully created in our directory containing our `Makefile`:

```
.  
├── build  
├── Makefile  
├── text_1  
└── text_2
```

Another run of `make`us tells us that `build`it is indeed up to date:

```
make build  
make: 'build' is up to date.
```

Now let’s delete `text_2`and relaunch `make`, it will recreate the latter and therefore the _build_ file :

```
rm text_2  
make build  
echo "World" > text_2  
cat text_1 text_2 > build
```

## Target`.PHONY`

There are targets that do not perform any “build”, those that clean the repository or display information. We will place these in the target, `.PHONY`for example:

```
.PHONY: view clean  
view:  
    [ -f build ] && cat build  
clean:  
    rm build text_1 text_2
```

Here’s what happens when you run the target `clean`:

```
make clean  
rm -f build text_1 text_2
```

The target `.PHONY`is used to avoid conflicts in file names: what would happen if a file `clean`existed? The target `clean`will never be launched since it has no dependencies and is therefore **always considered up-to-date**.

## Shell command in “recipes”

It is quite possible to use shell commands as instructions in our targets. It is thus possible to use built-in commands like `echo` or external commands like `mkdir`, `touch`, etc. It is also possible to use conditions, as you can see in the target `clean` of the previous example.

By default, `make` displays the instruction on standard output before executing it (and displays the result of it). It is possible not to display it by adding `@` in front:

```
.PHONY: echo  
echo:  
    echo "The command will be displayed"  
    @echo "Only the result is displayed"

```
And its execution:

make echo  
echo "The command will be displayed"  
The command will be displayed  
Only the result is displayed

By default, `make` uses `/bin/sh -c` to execute commands. It is possible to change the interpreter using the `SHELL` variable for the command and `.SHELLFLAGS` for the parameters. Here is an example for `bash`:

```
SHELL = /bin/bash  
.SHELLFLAGS = -c
```

## The variables

It is possible to use variables in `make`as we have seen above. It is customary to define variables in capital letters, but nothing is mandatory. There are four types of assignment:

- by reference via the `=`like sign `VAR = valeur`;
- by expansion with `:=`as `VAR := valeur`;
- conditional with `?=`as `VAR ?= valeur`, here the assignment will only take place if `VAR`has not been defined before;
- by concatenation with `+=`like `VAR += toto`.

To better understand the difference between the first two assignments, here is a Makefile example:

```
VAR = "Hello"  
REF = $(VAR)  
EXP := $(VAR)  
.PHONY: assign  
assign:  
    @echo "initial value: $(VAR)"  
    $(eval VAR = Bonjour)  
    @echo "new value: $(VAR)"  
    @echo "assign by reference \`VAR_2  = value\`: $(REF)"  
    @echo "assign by expansion \`VAR_2 := value\`: $(EXP)"
```

As you noticed, variables are used with the notation `$(VAR)`, it is also possible to use `${VAR}`.

Running the target note `assign`clearly shows that the variable assigned with `=` takes into account the modification of `VAR`via the function `eval`(we will talk about functions later), so it is a reference — in the manner of pointers — to it:

```
make assign   
initial value: Hello  
new value: Bonjour  
assign by reference `VAR_2  = value`: Bonjour  
assign by expansion `VAR_2 := value`: Hello
```

## Overload

It is possible to overload all the variables present in our `makefile`during its execution. This overload will take precedence over all the assignments of our `Makefile`:

```
make assign VAR="Guten tag"  
initial value: Guten tag  
new value: Guten tag  
assign by reference `VAR_2  = value`: Guten tag  
assign by expansion `VAR_2 := value`: Guten tag
```

Returning to our example, we can clearly see that even the assignment instruction `$(eval VAR = Bonjour)`no longer has any effect on the value of `VAR`.

## Specific variables

There are a basic set of defined variables to use in targets, here are some of them:

- `$@`contains the target name;
- `$<`contains the name of the first dependency;
- `$^`contains the list of all dependencies.

Let’s take our example with the `build`and add a target like below:

```
build: text_1 text_2  
    cat text_1 text_2 > build  
text_1:  
    echo "Hello" > text_1  
text_2:  
    echo "World" > text_2  
.PHONY: specific  
specific: text_1 text_2 build  
    @echo "target.....$@"  
    @echo "First dep..$<"  
    @echo "All deps...$^"
```

Running our target will cause, and (if necessary) `specific`to be built and display information like below:`buildtext_1text_2`

```
make specific  
target.....specific  
First dep..text_1  
All deps...text_1 text_2 build
```

# The functions

`make`has a set of functions that can be used in variables, or targets. There are functions to manipulate text, paths, execute shell commands, etc. The call to is done in the following form:

`$(function_name argument_1,argument_2)`

Let’s take the example of the function `subst`that allows you to substitute one pattern for another:

```
TEXT = hello world  
NEW_TEXT = $(subst hello, bonjour, $(TEXT))  
.PHONY: subst  
subst:  
    @echo $(NEW_TEXT)
```

The function takes three arguments:

1. the string to search for;
2. the one with which to substitute it;
3. the string to modify.

Note that macros can be used as arguments. Here is the output of the target one:

```
make subst :  
bonjour world
```

## Nesting functions

It is also possible to use functions as arguments, here is the example of `patsubst`which substitutes parts of path:

`$(patsub pattern, replacement, list_of_files)`

It is possible to use the function `wildcard`to list files based on a pattern given as an argument, here is an example:

```
FILES = $(patsubst text_%,my_text_%, $(wildcard text*))  
.PHONY: textfiles  
textfiles: build  
    @echo "origin: $(wildcard text*)"  
    @echo "files: $(FILES)"
```

Notice the use of `%`as a wildcard in the syntax of `Makefile`. However, it is necessary to use shell wildcards like `*`or `?`in target dependencies (to list files) and in the function `wildcard`.

## The messages

`make`has three functions to display information to the user:

1. `$(info message)`: simply displays message;
2. `$(warning message)`: displays the message preceded by the file and line number;
3. `$(error message)`: displays the message with file and line information and **stops execution of make** .

Here is an example:

```
.PHONY: messages  
messages:  
    $(info Information message)  
    $(warning Warning message)  
    $(error Error message)  
    $(info This message will not be displayed!)
```

And the result:

```
make messages  
Information message  
Makefile:54: Warning message  
Makefile:55: *** Error message.  Stop.
```

## Functions related to variables

It is possible to assign a function to a variable, it then becomes **a macro** . The modes by reference and by expansion with are still relevant. Here is an example of code to add after our file `Makefile`:

```
EXP_FILES := $(wildcard text*)  
REF_FILES = $(wildcard text*)  
.PHONY: macro  
macro: clean  
    @echo "EXP: $(EXP_FILES)"  
    @echo "REF: $(REF_FILES)"
```

Our target `macro`depends on the target `clean`which therefore deletes the files generated `text_1`and `text_2`retrieved by `wildcard`. The two actions of our target then display the variables, let's first launch our target `build` to ensure the presence of the files `text`, then the target `macro`. Here is the result:

```
make build  
[...]  
make macro  
rm -f build text_1 text_2   
EXP: text_1 text_2  
REF:
```

For `EXP_FILES`the result of the function `wildcard`is assigned to variable note. At this time, the files matching the pattern `text*`are still present.

For `REF_FILES`it is the function itself that is affected. It is therefore executed each time the variable is used. In the command `echo`the files matching the pattern `text*`no longer exist, so our function returns nothing.
# Conditional structures

It is possible to make a part of the `Makefile`accessible based on condition. Four types of conditions are available:

- equality via the command `ifeq (<val_1>, <val_2>)`that returns true if `<val_1>`is equal to `<val_2>`. Both elements can be macros or strings;
- non-equality with `ifneq (<val_1>,<val_2>)`which returns true if the two elements are different;
- the definition of a macro with `ifdef VAL`which returns true if `VAL` is defined (once the expansion is done);
- not defining a macro with `ifndef MACRO`.

Here is an example of using these structures:

```
TEST = hello  
DIST := $(shell lsb_release -i | awk -F ':\t' '{print $$2}')  
ifeq ($(DIST), Arch)  
MY_MESS := "By the Way"  
endif  
.PHONY: condition  
condition:  
ifndef MY_MESS  
    @echo "You are not using ArchLinux but $(DIST)"  
else  
    @echo $(MY_MESS)  
endif  
ifneq ($(TEST), hello)  
    @echo 'TEST has been modified'  
else  
    @echo "TEST has not been modified"  
endif
```

Here we use `ifeq`, `ifneq`and `ifndef`whether it is inside or even outside a target. We also have an example of using the function `shell`as an assignment to the variable `DIST`.

In targets, it is **imperative** to leave a tab before each instruction in the conditions otherwise the target will not work.

make condition  
By the Way  
TEST has not been modified

Now here is the result by changing the value of `TEST`at runtime:

make TEST=hallo DIST=Debian condition  
You are not using ArchLinux: Debian  
TEST has been modified
# In conclusion

We have seen in this article the most common notions of `make`, it is however a powerful tool whose operation cannot be summarized in an article. It proves useful in many situations that are not limited to development! In the coming weeks, I plan to present a few applications of what we’ve covered in this article for various uses.

https://medium.com/@talhakhalid101/mastering-make-a-beginners-guide-f969be23fcbf