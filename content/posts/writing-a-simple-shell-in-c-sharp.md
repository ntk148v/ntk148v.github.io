---
title: "Writing a Simple Shell in C Sharp"
date: "2023-08-20T23:15:34+07:00"
tags: ["tech", "csharp", "shell"]
draft: false
comment: true
---

In this post, we will write a minimalistic shell for UNIX(-like) operating systems in C# programming language. I create this for learning C# purpose.

Since its purpose demonstration (not feature completeness or even fitness for causual use), it has many limitations, including:

- Commands must be on a single line.
- Arguments must be separated by whitespace.
- No quoting arguments or escaping whitespace.
- No piping or redirection.

> Before we start, make sure you have .NET Core environment and knownledge.

## 1. What is a shell?

In computing, a [shell](<https://en.wikipedia.org/wiki/Shell_(computing)>) is a computer program that exposes an operating system's services to a human user or other programs. In general, operating system shells use either a command-line interface (CLI) or graphical user interface (GUI), depending on a computer's role and particular operation. It is named a shell because it is the outermost layer around the operating system.

Some examples:

- Bash
- Zsh
- Gnome Shell

In this article, I describe _a simple text-based, non-graphical shell_ with the basic functionality: give an input command and receive the output of this command.

```shell
# Input
$ ls

# Output
Documents   Downloads   Desktop
...
```

That's it!

## 2. Basic lifetime of a shell

Let's look at a shell from the top down. A shell does three main things its lifetime.

- **Initialize**: A typical shell would read and execute its configuration files. These change aspects of the shell's behavior. If you're familiar with bash shell, you may know `.bashrc` file, where you put the custom configuration.
- **Interpret**: The shell reads commands from stdin (which could be interactive, or a file) and executes them.
- **Terminate**: After its commands are executed, the seehll executes any shutdown commands, fres up and memory, and terminates.

But in our case, the shell will be simple as possible that there won't be any configuration files, and there won't be any shutdown command. So, we'll just call the input looping function and then terminate.

The basic structure looks like the following:

- Call our shell as `KShell`
- Print a prefix, just like Bash shell.
- Read the user input from keyboard.

```csharp
namespace KShell;

class KShell
{
    // Test it in Linux only
    private static string _currUser = Environment.UserName;
    private static string _currDir = Directory.GetCurrentDirectory();
    private static string _hostname = Environment.MachineName;

    static void Main(string[] args)
    {
        // Load config files, if any.

        // Run the input loop
        while (true)
        {
            Console.Write($"{_currUser}@{_hostname}:{_currDir}$ ");

            // Read the keyboard input
            string? input = Console.ReadLine();
            if (String.IsNullOrEmpty(input))
                continue;

            ExecCommand(input);
        }
    }

    static void ExecCommand(string input)
    {
        // Just print
        Console.WriteLine($"The input commmand: {input}");
    }
}
```

Build, run and we have the result, do you feel familiar?

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ cd
The input commmand: cd
```

## 3. Execute command

Now, we want to execute the entered command in `ExecCommand(string input)` function.

- First, remove the trailing blank spaces of the input, then parse the input separate the command and the arguments.
- Prepare the command.
- Clear the standard output, this is a trick to remove the prefix (we created before) from the output.
- Execute the command.
  - Starting processes is the main function of shells.
  - C# and .NET takes care all the difficult stuffs for us, but I recommend you to read about how shells start processes on Unix-like operating systems.
    - There are only two ways of starting processes on Unix. The first one (almost doesn't count) is by being `Init`. You see, when a Unix computer boots, its kernel is loaded. Once it is loaded and initialized, the kernel starts only one process, which is called Init. This process runs for the entire length of time that the computer is on, and it manages loading up the rest of the processes that you need for your computer to be useful.
    - Since most program aren't `Init`, that leaves only one practical way for processes to get started: the `fork()` system call. When this function is called, the operating system makes a duplicate of the process and starts them both running. The original process is called the "parent", and the new one is called the "child". `fork()` returns 0 to the child process, and it returns to the parent the process ID number (PID) of its child. In essence, this means that the only way for new processes is to start is by an existing one duplicating itself.
    - Typically, when you want to run a new process, you don't just want another copy of the same program - you want to run a different program. That's what the `exec()` system call is all about.
    - With the two system calls, we have the building blocks for how most program are run on Unix. First, an existing process forks itself into two separate ones. Then, the child uses `exec()` to replace itself with a new program. The parent process can continue doing other things, and it can even keep tabs on its children, using the system call `wait()`.

{{< figure class="figure" src="http://www.it.uu.se/education/course/homepage/os/vt18/images/module-2/fork-exec-exit-wait.png" caption="From it.uu.se: Operating systems" >}}

```csharp
static void ExecCommand(string input)
{
    // Split the input separate the command and the arguments
    string[] args = input.TrimEnd().Split(" ");

    // Execute command
    ProcessStartInfo startInfo = new ProcessStartInfo()
    {
        FileName = args[0],
        Arguments = string.Join("", args.Skip(1).ToArray()),
        RedirectStandardOutput = true,
        RedirectStandardError = true,
    };

    Process proc = new Process() { StartInfo = startInfo };

    // Clear the standard output
    // NOTE(kiennt26): This is a trick to remove the prefix from the command output
    proc.OutputDataReceived += (sender, e) => Console.WriteLine(e.Data);
    proc.Start();
    proc.BeginOutputReadLine();
    proc.WaitForExit();
}
```

Build and run the shell, enter your desired command.

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ ls
KShell
KShell.deps.json
KShell.dll
KShell.pdb
KShell.runtimeconfig.json

kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ ls -la
total 176
drwxrwxr-x 2 kiennt kiennt   4096 Thg 8  21 14:33 .
drwxrwxr-x 3 kiennt kiennt   4096 Thg 8  21 14:28 ..
-rwxr-xr-x 1 kiennt kiennt 142840 Thg 8  21 14:52 KShell
-rw-rw-r-- 1 kiennt kiennt    388 Thg 8  21 14:33 KShell.deps.json
-rw-rw-r-- 1 kiennt kiennt   6656 Thg 8  21 14:52 KShell.dll
-rw-rw-r-- 1 kiennt kiennt  10676 Thg 8  21 14:52 KShell.pdb
-rw-rw-r-- 1 kiennt kiennt    139 Thg 8  21 14:33 KShell.runtimeconfig.json

kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ pwd
/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0
```

It works nicely! It's starting to look like a real shell! But hold on, enter the super casual command - `cd`:

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ cd
Unhandled exception. System.ComponentModel.Win32Exception (2): An error occurred trying to start process 'cd' with working directory '/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0'. No such file or directory
...
```

Huh, something went wrong here. Why does the `cd` command not work? `cd` is not [real](https://stackoverflow.com/a/38776411) command, the functionality is a built-in command of the shell.

## 4. Shell Built-in Commands

Now, we will create some built-in commands. We have to modify the `ExecCommand` function: add a `switch` statement to the first argument (the command to execute) which is stored in `args[0]`.

### 4.1. `cd`

First, implement `cd`:

- If user enters `cd ~` or `cd` without arguments, the working directory is home.
- Change the working directory to the input.

```csharp
    static void ExecCommand(string input)
    {
        // Split the input separate the command and the arguments
        string[] args = input.TrimEnd().Split(" ");

        // Check for the built-in shell commands
        switch (args[0])
        {
            case "cd":
                BuiltInCD(args);
                break;
            case "#":
                // Handle the comment case
                break;
            default:
                // Execute command
                // ...
        }
    }

    /// <summary>
    /// Built-in cd - Change the shell working directory.
    /// Change the current directory to DIR. The default DIR is the value of the
    /// HOME shell variable.
    /// cd [dir]
    /// </summary>
    /// <param name="args"></param>
    static void BuiltInCD(string[] args)
    {
        string newWorkingDir;
        if (args.Length < 2)
        {
            // cd to home with empty path
            newWorkingDir = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
        }
        else if (args[1] == "~") // handle a special character
        {
            newWorkingDir = Environment.GetFolderPath(Environment.SpecialFolder.UserProfile);
        }
        else
        {
            newWorkingDir = args[1];
        }

        // Change the directory
        Directory.SetCurrentDirectory(newWorkingDir);
        _currDir = Directory.GetCurrentDirectory();
    }
```

### 4.2. `exit`

Similiarly, implement the `exit` command, it's quite simple.

```csharp
    // ...
    // ExecCommand
            case "exit":
                BuiltInExit(0);
                break;
    // ...

    /// <summary>
    /// Built-in exit - Exit the shell with a status of n. If n is omitted,
    /// the exit status is that of the last command executed.
    /// exit [n]
    /// </summary>
    /// <param name="exitCode"></param>
    static void BuiltInExit(int exitCode)
    {
        // TODO(kiennt26): Handle the given exit code, it should be in range 0-255
        Environment.Exit(exitCode);
    }
```

### 4.3. `which`

`which` returns the pathnames of the files (or links) which would be executed in the current environment. It does this by searching the PATH for executable files matching the file names of the arguments.

```csharp
// Get the PATH environment variable for a list of directories.
    private static string[] _path = Environment.GetEnvironmentVariable("PATH").Split(":");
```

Create a function that searches the PATH for executable files matching the file names of the arguments.

```csharp
    static List<string> SearchInPath(string executable)
    {
        List<string> pathNames = new List<string>();
        // string[] pathNames = new string[0];
        foreach (string p in _path)
            pathNames.AddRange(Directory.GetFiles(p, executable));

        return pathNames;
    }
```

Then, create `BuiltInWhich` function to print out the result. If there is no matching executable file, returns nothing.

```csharp
    // ExecCommand
            case "which":
                BuiltInWhich(args);
                break;


    /// <summary>
    /// Built-in which - locate a command
    /// which returns the pathnames of the files (or links) which would be executed in the current environment.
    /// It does this by searching the PATH for executable files matching the file names of the arguments.
    /// </summary>
    /// <param name="args"></param>
    static void BuiltInWhich(string[] args)
    {
        if (args.Length < 2)
            return;
        foreach (string executable in args.Skip(1).ToArray())
        {
            foreach (string p in SearchInPath(executable))
                Console.WriteLine(p);
        }
    }
```

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ which ls
/usr/bin/ls
/bin/ls
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ which pwd
/usr/bin/pwd
/bin/pwd
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ which cat grep
/usr/bin/cat
/bin/cat
/usr/bin/grep
/bin/grep
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$

```

### 4.4. `help`

A help page always necessary, let's create one. Logic is simple:

- If user enter `help` command alone, returns a general help.
- If user enter `help <built-in>`, returns the command specified help.

```csharp
    // ExecCommand
            case "help":
                BuiltInHelp(args);
                break;

    static void BuiltInHelp(string[] args)
    {
        string help;
        if (args.Length < 2)
        {
            help = @"
KShell aka. Kien's Shell, written in C#.

    Type program names and arguments, and hit <enter>.
    These shell commands are defined internally.  Type `help` to see this list.
    Type `help name` to find out more about the function `name'.

    cd [dir]
    exit [n]
    which filename ...
    help";
        }
        else
        {
            switch (args[1])
            {
                case "cd":
                    help = @"
cd: cd [dir]

    Change the shell working directory.

    Change the current directory to 'dir'. The default 'dir' is the value of the user's home directory.";
                    break;
                case "exit":
                    help = @"
exit: exit [n]

    Exit the shell.

    Exits the shell with a status of 'n'.";
                    break;
                case "which":
                    help = @"
which: which filename ...

    Locate a command.

    which returns the pathnames of the files (or links) which would be executed in the current environment.
    It does this by searching the PATH for executable files matching the names of the arguments.
";
                    break;
                default:
                    help = @"
KShell aka. Kien's Shell, written in C#.

    Type program names and arguments, and hit <enter>.
    These shell commands are defined internally.  Type `help` to see this list.

    cd [dir]
    exit [n]
    help";
                    break;
            }
        }

        Console.WriteLine(help);
    }
```

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ help cd

cd: cd [dir]

    Change the shell working directory.

    Change the current directory to 'dir'. The default 'dir' is the value of the user's home directory.
```

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ help

KShell aka. Kien's Shell, written in C#.

    Type program names and arguments, and hit <enter>.
    These shell commands are defined internally.  Type `help` to see this list.
    Type `help name` to find out more about the function `name'.

    cd [dir]
    exit [n]
    which filename ...
    help
```

## 5. Improvement

### 5.1. Handle exception

Entering the wrong command, and a long stacktrace returns. It makes nonsense for the end user. The end user just need a message like: "command not found". Just it.

We will wrap the main input loop in try/catch.

```csharp
    static void Main(string[] args)
    {
        // Load config files, if any.

        // Run the input loop
        while (true)
        {
            try
            {
                Console.Write($"{_currUser}@{_hostname}:{_currDir}$ ");

                // Read the keyboard input
                string? input = Console.ReadLine();
                if (String.IsNullOrEmpty(input))
                    continue;

                ExecCommand(input);
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
                // 0 - Success
                // 1 - Fail
                BuiltInExit(1);
            }
        }
    }
```

Looks much better now:

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ wrongcommand
An error occurred trying to start process 'wrongcommand' with working directory '/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0'. No such file or directory
```

### 5.2. Handle command not found

The exception's message is still not clear enough for the end user. We can check if the command is existing before execute it. Remember `search3`

```csharp
    // ExecCommand
            default:
                // Check if args[0] is an executable file
                if (SearchInPath(args[0]).Count < 1)
                {
                    throw new Exception($"{args[0]}: command not found");
                }

                // ...
```

```shell
kiennt@kiennt-ROG-Strix-G513IH-G513IH:/home/kiennt/Workspace/github.com/ntk148v/Solution1/KShell/bin/Debug/net6.0$ wrongcommand
wrongcommand: command not found

```

## 6. Wrap up

I hope you enjoyed it. I think, when you understand the concepts behind it, it's quite simple (especially with high-level programming language like C#).

Feel free to extend the shell with new features.

This post is highly inspired by:

- [Writing a simple shell in Go](https://simjue.pages.dev/post/2018/07-01-go-unix-shell/)
- [Write a shell in C](https://brennan.io/2015/01/16/write-a-shell-in-c/)

The code for `KShell` is available on [Github](https://github.com/ntk148v/KShell/tree/master).
