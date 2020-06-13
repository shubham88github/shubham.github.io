---
title: "Linux Shell Scripting"
excerpt_separator: "<!--more-->"
categories:
  - bash
tags:
  - bash
  - linux
  - programming
  - linux-tutorial
---

In linux when you open a terminal, youre presented with a terminal. Over here you can only use shell commands, or BASH. The history of shells goes all the way back to the good'ol days when poeple were using UNIX OS and it had Sh- Bourne Shell. Eventually we kept seeing more and more shells out there.

#### Types of Shells
+ Sh-Bourne Shell
+ Csh-C Shell
+ Ksh-Korn Shell
+ Tcsh-enhanced C Shell
+ Bash-GNU Bourne Again Shell ( What we get by default on most linux distributions )
+ Zsh-extension to Bash, Ksh, and Tcsh ( What i like to use combined with oh-my-zsh )
+ Pdksh-extension to KSH

Bash and Zsh has more features than the others. I like to use zsh because of its feature rich experience. Once combined with oh-my-zsh it looks quite appealing as well.

#### Purpose of the shell
The shell is used to execute commands on the system. These tasks vary from reading text files, signal handling, handling the execution of programs, etc.

#### Some beginner commands

```shell
ls     # check the contents of a directory
pwd     # gives the full path to the current directory
mkdir files     # creates a new directory called files
cd files     # will change the current working directory to the files directory
touch hello.sh     # this will create an empty file called hello.sh
cp hello.sh world.sh     # this will create an exact copy of hello.sh and save it as world.sh
mv hello.sh hey.sh    # this will rename the file / move the file ( in this case rename hello.sh to hey.sh )
```

#### First bash script

Now lets try create our first bash script, hello.sh. We will be using a text editor called nano. Since its beginner friendly. `nano hello.sh`. Type the contents below into the file. press ctrl + o to write the changes to a file and then ctrl + x to exit the program.
```shell
#!/bin/bash
echo "welcome to the world of bash"
CURRENT_DATE=$( date )
CURRENT_DIRECTORY=$( pwd )
CURRENT_DIRECTORY_CONTENTS=$( ls )

echo "Current date is $CURRENT_DATE"
echo "Current directory is $CURRENT_DIRECTORY"
echo "Contents of the current directory $CURRENT_DIRECTORY_CONTENTS"
```
Now you have written your first shell script! congratulations! Lets try execute the script. `bash hello.sh`. That should give you an output that looks something like.
```
$ bash hello.sh
welcome to the world of bash
Current date is Sun Apr 22 14:25:42 AEST 2018
Current directory is /home/eshan/files
Contents of the current directory hello.sh
hey.sh
world.sh
```
The lines starting with # is treated as comments, except the line `#!/bin/bash`. This is an exception where its used to identify the program which will be used to execute the script. The `echo` is used to print contents to the screen. `VARIABLE_NAME=3` is used to assign variable. Be careful with the no space between the equal sign and the variable name. That will cause an error if you have space between them. `VARIABLE=$( COMMAND )` is used to execute a COMMAND and then the output of the command is assigned to the VARIABLE. So you can use the `echo` to print those contents out, or do other processing on these. I have gotton the current date, current working directory and contents of the current directory in a variable. And i am printing those out to the screen. Hope this helps anyone starting out with bash!

