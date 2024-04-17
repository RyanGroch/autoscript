# Autoscript

This script automatically types commands into a terminal. It is designed to help computer science students at Harper College prepare their assignments for submission. Students at Harper are often required to use the `script` Unix command to record the inputs and outputs of their programs. Students manually run their programs several times, and doing so requires a reasonable amount of typing. This program does the typing for students so that they can speed up their workflows.

Autoscript was designed to run on Ubuntu Server. Users should invoke this script over SSH rather than run it locally on their own computers.

## Dependencies

Two command line utilities are necessary for this script to run properly:

- [tmux](https://github.com/tmux/tmux/wiki), a utility that allows users to open pseudo-terminals.
- [yq](https://github.com/mikefarah/yq), a utility for parsing [YAML](https://yaml.org/spec/1.2.2/) files.

The installation commands are below:

```sh
sudo apt install tmux
sudo snap install yq
```

It is worth mentioning that `tmux` should be installed by default on Ubuntu Server.

## Usage

The primary way to use Autoscript is to pass in a configuration file, like this:

```sh
autoscript autoscript_config.yml
```

Autoscript will use the `tmux` module to open a pseudo-terminal. It will then type all commands specified in the configuration file.

Configuration files must have valid [YAML](https://yaml.org/spec/1.2.2/) syntax and must have a `commands` label containing a sequence of the user's desired inputs. The following is a minimal "Hello World" file:

```yml
commands:
  - 'echo "Hello World"'
  - "exit" # Closes the pseudo-terminal
```

If you pass the above file into Autoscript, the following will happen in order:

1. The user will see a confirmation message and will be prompted to press `Enter`.
2. A pseudo-terminal will open.
3. Autoscript will type `echo "Hello World"` into the pseudo-terminal and execute it.
4. Autoscript will type `exit` into the pseudo-terminal and execute it. This will close the pseudo-terminal.

### Using delays

Autoscript doesn't know whether the commands it typed were successful. It also doesn't know whether the commands it typed have finished executing. As such, it is necessary to instruct Autoscript to pause for a number of milliseconds after it has finished typing each command.

Every configuration file may contain a `default_delay` label, specifying how many milliseconds Autoscript should pause after typing each command. The "Hello World" file from above can be rewritten like this:

```yml
default_delay: 500
commands:
  - 'echo "Hello World"'
  - "exit" # Closes the pseudo-terminal
```

The `default_delay: 500` specifies that Autoscript should pause for 500 milliseconds (or 0.5 seconds) after entering each command. In actuality, Autoscript uses the value of `500` for `default_delay` when the user does not provide a `default_delay` themselves. As such, the above addition doesn't change anything. If we had written `default_delay: 1000` instead, then Autoscript would pause for a full second after typing each command.

Some commands take longer than others to execute. For instance, compiling a complex C++ program is likely to take much longer than `echo "hello world"`. Autoscript accounts for this by allowing the user to override `default_delay` for individual commands. Suppose that we are using `g++` to compile a program, and we want Autoscript to pause for a full 5 seconds while this happens. We could make the following addition to the file above:

```yml
default_delay: 500
commands:
  - 'echo "Hello World"'
  - "g++ program_name.cpp": 5000 # Pause for 5 seconds
  - "exit" # Closes the pseudo-terminal
```

Autoscript will pause for 5 seconds after entering `g++ program_name.cpp`. This way, we give the compiler time to do its job before Autoscript tries to type the next command. We can use the `<COMMAND>: <DELAY>` syntax for as many commands as we would like, but we must do so each time we would like Autoscript to pause for a length of time that differs from `default_delay`.

### Generating Configuration Files

Autoscript allows the user to automatically generate boilerplate for configuration files. Running the below command will create a new file with the name `autoscript_config.yml`, assuming that no such file or directory already exists:

```sh
autoscript --config
```

This is the recommended way to use Autoscript. The user can edit the configuration file to suit their specific program.

If the user wishes to provide a specific name for the new configuration file, they may use the following command instead:

```sh
autoscript --config name_of_new_configuration_file
```

### Using the Autoscript Example

Autoscript comes with a built-in demo so that users can immediately see it in action. To try it out, enter the following command:

```sh
autoscript --example
```

This will create a directory called `autoscript_example`, assuming that no such file or directory already exists. You can also choose a different name for the example directory like this:

```sh
autoscript --example name_of_new_directory
```

The example directory contains a C++ lab similar to what introductory computer science students at Harper College might write. You can use the configuration file inside the directory to immediately script the C++ program as is.

The example configuration file invokes scripts specific to Harper's server. As such, it cannot be expected to produce a desirable result on machines without those scripts installed.
