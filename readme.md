# Autoscript

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
  - [Installation for Server Administrators](#installation-for-server-administrators)
  - [Installation for Harper Students](#installation-for-harper-students)
    - [Installing yq](#installing-yq)
    - [Installing the Autoscript File](#installing-the-autoscript-file)
    - [Adding Autoscript and yq to your PATH](#adding-autoscript-and-yq-to-your-path)
  - [Installation on Other Devices](#installation-on-other-devices)
- [Usage](#usage)
  - [Configuration File Basics](#configuration-file-basics)
    - [Using Delays](#using-delays)
    - [Syntax Notes](#syntax-notes)
  - [Practical Configuration Files](#practical-configuration-files)
    - [Program Inputs](#program-inputs)
  - [Generating Configuration Files](#generating-configuration-files)
  - [Using the Autoscript Example](#using-the-autoscript-example)

## Introduction

This script automatically types commands into a terminal. Its purpose is to help computer science students at Harper College prepare their assignments for submission.

Some computer science courses at Harper require students to use the `script` Unix command to record the inputs and outputs of their programs. Students run their programs several times, which involves a reasonable amount of typing. [Jason James](http://craie-programming.org/), professor of computer science at Harper College, [describes this process in detail on his website](http://craie-programming.org/lab/basics.html#script). Autoscript removes much of the human error from this process and allows students to speed up their workflows.

My intent is for Autoscript to run on the Ares server at Harper College. Students should invoke this script over SSH rather than run it locally on their own computers. It is possible, though likely impractical, to run Autoscript on other devices. See [Installation on Other Devices](#installation-on-other-devices) for more detail.

You may find this script useful if you are a Harper student taking a computer science course that involves the process described above. Please ask your instructor whether it is acceptable to use Autoscript if you are interested in doing so.

## Installation

With any luck, Autoscript will one day be installed globally on Harper's Ares server. This way, students will be able to use Autoscript by default without worrying about installation. You can check whether Autoscript is installed by entering `autoscript` into your terminal when you SSH to Ares. If it is installed, then you will see a list of options. Otherwise, you will see output that reads `autoscript: command not found`.

Even if Autoscript is not installed globally, students may still run a personal version of Autoscript on Ares. See [Installation for Harper Students](#installation-for-harper-students) for more detail.

### Installation for Server Administrators

If you are an administrator of Harper's Ares server and intend to install Autoscript globally, then you may find this section to be useful.

Autoscript requires two command-line utilities to run properly:

- [tmux](https://github.com/tmux/tmux/wiki), a utility that allows users to open pseudo-terminals.
- [yq](https://github.com/mikefarah/yq), a utility for parsing [YAML](https://yaml.org/spec/1.2.2/) files.

Fortunately, `tmux` is installed by default on Ubuntu Server. Therefore, it should only be necessary to install `yq`.

The [yq GitHub Page](https://github.com/mikefarah/yq#install) offers several installation methods, but some of these involve making requests to endpoints on GitHub with either `curl` or `wget`. I find that GitHub rejects these requests when they come from the Ares server. Therefore, you will likely find more success installing `yq` with either of the two methods below:

1. Run `sudo snap install yq`. This is the faster and easier of the two methods. However, some consider Snaps to be unpreferable.
2. Download the Linux/x86 `yq` binary from [here](https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64). Make sure you rename the file to `yq`; Autoscript won't be able to recognize it otherwise. You can then use FTP or SFTP to transfer the binary to the Ares server. The `yq` binary can presumably go in either `/usr/bin` or `/usr/local/bin`; use whichever one you find to be more appropriate. You will need to designate `yq` as executable with a command such as `chmod +x yq`.

After installing `yq`, the next step is to install `autoscript`. Start by running the following:

```sh
sudo nano /usr/local/bin/autoscript
```

Copy and paste the contents of the `autoscript` file from this repository directly into your `nano` editor. Save the file and close `nano`. Don't forget to run `sudo chmod +x /usr/local/bin/autoscript` to designate `autoscript` as executable.

Afterwards, any user should be able to enter the `autoscript` command and see a list of available options.

### Installation for Harper Students

Ideally, you shouldn't need to install Autoscript yourself; it should eventually be installed globally on the Ares server. Check by typing `autoscript` into your SSH terminal. If the output is anything other than `autoscript: command not found`, then Autoscript is already installed.

However, there are a few reasons for which you might want to install a personal version of Autoscript on Ares:

1. If Autoscript is not installed on Ares.
2. If an outdated version of Autoscript is installed on Ares, and you wish to use an updated one.
3. If you wish to modify Autoscript to better suit your personal needs. You won't be able to edit the global installation of Autoscript, so you will need a personal version.

If you find that any of these cases apply, then read on.

#### Installing yq

Autoscript relies on a program called `yq`, and cannot run properly without it. Autoscript's configuration files use a format called YAML, and `yq` parses YAML files. You can check whether `yq` is installed by typing `yq` into your SSH terminal. If it is installed, then you will see some text describing basic usage. If it is not installed, then you will see text which looks something like this:

```
Command 'yq' not found, but can be installed with:

snap install yq
Please ask your administrator.
```

If `yq` is already globally installed, then you won't need to install it yourself. In that case, feel free to skip to the [Installing the Autoscript File](#installing-the-autoscript-file) section.

Unfortunately, the installation process is not straightforward. You _cannot_ use `snap install yq` yourself; only the server administrator can do that. As such, the only way for you to install `yq` is to download the `yq` binary and transfer it to the server. Carefully follow the steps below:

1. Download the official`yq` release from [this link](https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64).
2. Rename the file you just downloaded to `yq`.
3. Open an SFTP client such as FileZilla or WinSCP. Connect to Harper's Ares server and transfer your new `yq` file to your home directory on the server.
4. SSH to the Ares server. Many people use [PuTTY](https://www.putty.org/) or a similar program for this.
5. Move your new `yq` file to its own separate directory. You can do this automatically by running the block of code below in your terminal. I will assume we are calling the new directory `autoscript_files`.

```sh
# creates the new directory if it does not already exist
mkdir ~/autoscript_files

# moves yq into your new directory;
# only works if yq was originally in your home directory to begin with
mv ~/yq ~/autoscript_files/yq
```

6. Designate `yq` as an executable file. You can do this by running the following block of code in your terminal:

```sh
chmod u+x ~/autoscript_files/yq
```

7. Check that your installation was successful by running the command below. You should see a body of text describing `yq`'s basic usage.

```sh
~/autoscript_files/yq
```

If you followed all of these steps carefully, then you've finished installing `yq`. At this point, you can move on to [Installing the Autoscript File](#installing-the-autoscript-file). Note that you do not need to understand how to use `yq` yourself; it just needs to be present and accessible.

#### Installing the Autoscript File

Make sure that `yq` is already installed before beginning this step. If `yq` isn't installed globally on Ares, then make sure that you follow the steps in the section above.

To install Autoscript, carefully follow the steps below in order:

1. Connect to the Ares server via SSH. Many people use [PuTTY](https://www.putty.org/) or a similar program for this.
2. Make sure that the `autoscript_files` directory described in the [Installing yq](#installing-yq) section exists. Note that the `autoscript_files` directory will eventually hold both your `yq` file (if applicable) and your `autoscript` file.

```sh
# Create `autoscript_files` if it does not already exist
mkdir ~/autoscript_files
```

3. Create a file in `autoscript_files` called `autoscript` and open it with a text editor. The easiest way to do this is with the following command:

```sh
nano ~/autoscript_files/autoscript
```

4. Outside of your terminal, copy the entire contents of the `autoscript` file in this GitHub repository. Click on the file to open it. Use your mouse to highlight the entire contents of the file, and press `CTRL+C` to copy the contents onto your computer's clipboard.
5. Use `CTRL+SHIFT+V` (or right click and select 'paste') to paste the contents of your clipboard directly into your SSH terminal.
6. With your terminal open, press `CTRL+O` (in `nano`) to save the file.
7. Press `CTRL+X` to exit `nano`.
8. In your terminal, enter the following command to designate your new `autoscript` file as executable:

```sh
chmod u+x ~/autoscript_files/autoscript
```

9. Test that your installation was successful by running the following command below. If your installation was successful, you should see a list of instructions describing basic usage:

```sh
~/autoscript_files/autoscript
```

Assuming that you have completed the above steps successfully, then you are ready to move on to the final step described in the section below.

#### Adding Autoscript and yq to your PATH

This is the final step to installing Autoscript. You need to make sure that both the `autoscript` file and the `yq` file (if `yq` is not installed globally) are added to your PATH variable.

Make sure that you have followed the steps in the previous two sections carefully, and then type the following command into your SSH terminal:

```sh
PATH="~/autoscript_files:$PATH"
```

After running this command, you should be able to type the `autoscript` command from any directory and be able to see Autoscript's usage instructions printed into the terminal. At this point, you will be able to use Autoscript just as if it were installed globally.

Unfortunately, you will need to redo this step each time that you start an SSH session. Changes that you make to your `PATH` variable are not preserved between SSH sessions.

### Installation on Other Devices

As previously mentioned, I intended for Autoscript to run on Harper's Ares server. If you are trying to install Autoscript on any other device, chances are that you are either experimenting or have found a different use for Autoscript that I did not originally account for. If you are a Harper student who wants to use Autoscript for their coursework, then the [Installation for Harper Students](#installation-for-harper-students) section will be more useful to you.

It is perfectly feasible to install Autoscript on other devices. The process will vary based on your specific device, so I won't go through any steps in detail. There are, however, a some points worth considering if you wish to pursue this option:

- Autoscript is a Bash (**B**ourne **A**gain **SH**ell) script. I have not tested whether other shell languages such as ZSH can run Autoscript. If you are trying to run Autoscript on a Mac, I'd recommend installing Bash first and using it when running Autoscript.
- Autoscript requires both `tmux` and `yq` to run properly. No matter where you are running it, you must have those two utilities installed and added to your PATH variable.
  - Every operating system will differ in terms of package managers. As such, installation for these programs will differ widely based on operating system.
  - Make sure to install a version of `yq` that is appropriate for both your CPU architecture and your operating system. The link to download `yq` that I've provided in other sections is specific to x86 CPU processors paired with a Linux operating system. You may need to choose a different installation method from the [yq GitHub page](https://github.com/mikefarah/yq/#install).
- If you are using Windows, I'd recommend using either [Cygwin](https://www.cygwin.com/) or [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/). Either of these will allow you to run a Linux environment on your Windows machine. It will likely be much easier to run Autoscript on Linux than in other environments.
- Some of the boilerplate files generated by `autoscript --example` and `autoscript --config` invoke scripts specific to Harper's Ares server. The boilerplate files will not work properly unless you have those scripts installed locally on your machine. While these scripts are not necessary for Autoscript to work in general, they are necessary if you are trying to replicate the environment on the Ares server. [Jason James has these scripts available on his website](http://craie-programming.org/C++/free.compilers.html), and the ones that Autoscript references are as follows:
  - `show-code`
  - `CPP`
  - `script-print`
- Harper's Ares server runs quite fast. If you are running Autoscript on a slower device, then you may need to include longer delays than you would on Ares.

## Usage

The primary way to use Autoscript is to pass in a configuration file, like this:

```sh
autoscript autoscript_config.yml
```

In this context, `autoscript_config.yml` is the configuration file that you are using to run Autoscript. Configuration files must have valid [YAML](https://yaml.org/spec/1.2.2/) syntax and must have a `commands` label containing a sequence of your desired inputs. In the configuration file, you can include all of the commands that you would typically use to script your program. For instance, you might include `script`, `pwd`, `show-code`, and so on. Again, Jason James describes the manual process in detail [on his site](http://craie-programming.org/lab/basics.html#script).

Once you pass in a valid configuration file, Autoscript will prompt you for confirmation. After you confirm, Autoscript will open a pseudo-terminal and type all commands specified in the configuration file.

If you wish to interrupt Autoscript after it has opened the pseudo-terminal, then you can press `CTRL + b` followed by `d`. Doing so causes you exit the pseudo-terminal and return to your main terminal. You can then press `CTRL + c` to stop Autoscript, which always runs from the main terminal.

### Configuration File Basics

Configuration files must have valid [YAML](https://yaml.org/spec/1.2.2/) syntax and must have a `commands` label containing a sequence of the user's desired inputs. The following is a minimal "Hello World" file:

```yml
# autoscript_config.yml

commands:
  - "echo 'Hello World'"
```

If you pass the above file into Autoscript, the following will happen in order:

1. The user will see a confirmation message and will be prompted to press `Enter`.
2. A pseudo-terminal will open.
3. Autoscript will type `echo 'Hello World'` into the pseudo-terminal and execute it.

Notice that Autoscript does not close the pseudo-terminal after it finishes its job. You probably won't want to keep the pseudo-terminal open, though; you cannot start Autoscript again until you return to the main terminal. To instruct Autoscript to close the pseudo-terminal automatically, you can add an `exit` command at the end of your file:

```yml
commands:
  - "echo 'Hello World'"
  - "exit" # Closes the pseudo-terminal
```

#### Using delays

Autoscript doesn't know whether the commands it typed have finished executing. As such, it is useful to instruct Autoscript to pause for a number of milliseconds after it has finished typing each command. This way, commands that take a while to run will have a sufficient amount of time to finish executing before Autoscript moves on to the next command.

Every configuration file may contain a `default_delay` label, specifying how many milliseconds Autoscript should pause after typing each command. The "Hello World" file from above can be revised to this:

```yml
default_delay: 500
commands:
  - "echo 'Hello World'"
  - "exit" # Closes the pseudo-terminal
```

The `default_delay: 500` specifies that Autoscript should pause for 500 milliseconds (or 0.5 seconds) after entering each command. If the user does not provide a `default_delay` themselves, then `default_delay` is automatically set to 0.

Some commands typically take longer than others to execute. For instance, compiling a complex C++ program is likely to take longer than `echo 'Hello World'`. Autoscript accounts for this by allowing the user to override `default_delay` for individual commands.

Suppose that we are using `CPP`, a Harper-specific script, to compile a C++ program. Suppose that we also want Autoscript to pause for a full 5 seconds while this happens. We could make the following addition to the file above:

```yml
default_delay: 500
commands:
  - "echo 'Hello World'"
  - "CPP program_name.cpp": 5000 # Pause for 5 seconds
  - "exit" # Closes the pseudo-terminal
```

Autoscript will pause for 5 seconds after entering `CPP program_name.cpp`. This way, we give the compiler time to do its job before Autoscript tries to type the next command. We can use the `COMMAND: DELAY` syntax for as many commands as we would like, but we must do so each time we would like Autoscript to pause for a length of time that differs from `default_delay`.

#### Syntax Notes

Autoscript's YAML parser, `yq`, can be somewhat finicky about syntax. In order to avoid errors, you should consider the following rules to be mandatory:

- Each command under the `commands` label should be preceeded by a hyphen. All hyphens should be indented equally, and there should be a space between the hyphen and the beginning of the command.
- There should be _not_ be a space between `default_delay` and the colon that follows it. However, there _must_ be a space between the colon and the integer that follows it.
- The same point applies to individual commands with delays; there must be _not_ between the command and the colon, but there _must_ be a space between the colon and the integer representing the delay.
- The `default_delay` and `commands` labels should not be indented at all.
- The colon following the `commands` label should not be followed by anything on the same line other than spaces.

Otherwise, many syntax considerations are optional. For instance, the use of quotation marks is generally not required.

Additionally, note that line comments begin with a `#` sign. Autoscript will ignore comments entirely.

### Practical Configuration Files

None of the configuration files we've seen up to this point are of much practical value. After all, the reason Autoscript exists is to help students script their assignments. This is going to require configuration files that are more complicated than what we've seen thus far. Consider the example file below:

```yml
default_delay: 0
commands:
  - "script": 500
  - "pwd"
  - "cat progname.info" # this only works if you have created an information file

  - "show-code progname.cpp": 2000 # repeat for each of your .h and .cpp files if using libraries

  - "CPP progname.cpp": 1500 # list all .cpp filenames if using libraries

  - "./progname.out" # run 3-5 tests of your program to show a good sample of your debugging efforts

  - "cat progname.tpq" # this only works if you have created a Thought-Provoking Questions file

  - "exit" # ends the "script" command

  - "script-print Last-First-Progname": 2000

  - "exit" # exits pseudo-terminal
```

You may have noticed that structure of the file above closely resembles [the scripting process that Jason James describes on his website](http://craie-programming.org/lab/basics.html#script). In fact, I copied and pasted most of the lines above directly from the page I just linked to.

This is far closer to how a real configuration file should look if you are a computer science student at Harper. In fact, this file would probably work fine as-is for scripting a simple "Hello world" C++ program. Of course, some minor adjustments may be necessary; for instance, you'd want to change `progname` to the actual name of your program, and you might remove the `cat progname.tpq` line if you don't have a TPQ file.

You might also have noticed that I've added delays on commands such as `script`, `show-code`, `CPP`, and `script-print`. The reason for this is that I've found these commands sometimes take longer than others to run. In practice, you should feel free to adjust all delays as you see fit.

#### Program Inputs

Most of the programs that you'll write for your courses will take user input. In addition to having Autoscript run shell commands, you can also have Autoscript pass input into your programs as they are running. Consider the example C++ program below:

```cpp
// autoscript_example.cpp

#include <iostream>

using namespace std;

int main()
{
    double num1, num2;

    cout << "\nEnter your first number: ";
    cin >> num1;

    cout << "Enter your second number: ";
    cin >> num2;

    cout << "\nThe sum is " << (num1 + num2) << "\n\n";

    return 0;
}
```

This program will prompt the user for input twice. Fortunately, Autoscript is capable of seamlessly typing input into programs. Suppose that we want to run the above program with a "first number" of `1` and a "second number" of `2`. Our terminal might look something like this as we run our program:

```
Enter your first number: 1
Enter your second number: 2

The sum is 3
```

We can easily write a configuration file that instructs Autoscript to compile our program and automatically run it with the inputs specified above. Adding program inputs to our configuration files isn't really any different from adding any other command. This is what our configuration file might look like:

```yml
default_delay: 0
commands:
  - "CPP autoscript_example.cpp": 1500 # Compile the program

  - "./autoscript_example.out" # Run the program

  - "1" # Enter the value 1 as "first number"
  - "2" # Enter the value 2 as "second number"

  # We can assume that the program has finished running at this point

  - "exit" # Closes the pseudo-terminal
```

For your real configuration file, you will of course want to include the other required commands such as `script`, `pwd`, and so on. You will likely also want to run your program more than once with a variety of inputs. If you were taking a class with Jason James and `autoscript_example` were a lab assignment, your final configuration file would probably look something like this:

```yml
default_delay: 0
commands:
  - "script"
  - "pwd"
  - "cat autoscript_example.info"
  - "show-code autoscript_example.cpp": 2000
  - "CPP autoscript_example.cpp": 1500

  # Test run #1 - expected output is 3
  - "./autoscript_example.out"
  - "1"
  - "2"

  # Test run #2 - expected output is 6
  - "./autoscript_example.out"
  - "10"
  - "-4"

  # Test run #3 - expected output is 8.32
  - "./autoscript_example.out"
  - "2.87"
  - "5.45"

  - "cat autoscript_example.tpq"

  - "exit" # ends the "script" command

  - "script-print Last-First-AutoscriptExample": 2000

  - "exit" # exits pseudo-terminal
```

This configuration file could generate an output PDF suitable for turning in.

### Generating Configuration Files

Autoscript allows you to automatically generate boilerplate for configuration files. Running the below command will create a new file with the name `autoscript_config.yml`, assuming that no such file or directory already exists:

```sh
autoscript --config
```

This is the recommended way to use Autoscript. You can edit the configuration file to suit your specific program.

If you wish to provide a specific name for the new configuration file, you may use the following command instead:

```sh
autoscript --config name_of_new_configuration_file
```

### Using the Autoscript Example

Autoscript comes with a built-in demo so that you can immediately see it in action. To try it out, enter the following command:

```sh
autoscript --example
```

This will create a directory called `autoscript_example`, assuming that no such file or directory already exists. You can also choose a different name for the example directory like this:

```sh
autoscript --example name_of_new_directory
```

The example lab is identical to the one shown in the [Program Inputs](#program-inputs) section of this document. You can use the configuration file inside the directory to immediately script the C++ program as is.
