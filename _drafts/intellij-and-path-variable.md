
tags: [zsh, bash, intellij, env variables, linux] #check

----

Recently I encountered an issue with how Intellij reads environment variables. In one project I was helping Maven build was using `npm` command to build some frontend files. Unfortunately, when I started Intellij IDEA using a desktop shortcut (in Ubuntu) and ran Maven build inside the IDE it didn't see my `PATH` variable defined in configuration file (neither `zsh` nor `bash`).

How to solve that?

Starting the IDE from shell is one option, the easiest one. But it can be really annoying. Imagine you already have a session running, which was started using .desktop file, and you notice that your variables are not there and you have to start it again, this time using console. There must be a better way.

As it [turns out](https://youtrack.jetbrains.com/issue/IDEABKL-7589) IDEA starts using login shell. It means that during startup files like `.bashrc` or `.zshrc` are not read. Instead, you can put your configuration in `~/.profile` and relogin (log out and log in again, or even better restart your machine). It's kind of surprising to me how it works, although it might be connected to how Gnome starts programs, not necesarily IDEA startup scripts. Apparently it does not read `~/.zprofile`, even though I use `zsh`. Also, it ignores `~/.bash_profile`, even though it should [take precedence](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html) over `~/.profile` during login shell startup. Kind of a mess, isn't it?

Anyway, setting your variables in `~/.profile` works. But, what if you don't want to reconfigure all your startup scripts to workaround one problem?

What you can do is to modify `.desktop` shortcut `~/.local/share/applications/jetbrains-idea.desktop` and replace line

```bash
Exec="/home/amroz/Dev/ide/idea-IU-183.4284.148/bin/idea.sh" %f
```

with
```bash
zsh -c "/home/amroz/Dev/ide/idea-IU-183.4284.148/bin/idea.sh" %f
```

replace `zsh` with `bash` if you don't use `zsh`.

From manual:
```
-c Take  the  first  argument  as a command to execute, rather than reading commands from a script or standard input.  If any further arguments are given, the first one is assigned to $0, rather than being used as a positional parameter.
```

In essence, it would ....


A word of warning here, though. It might be overwritten if the `.desktop` file is regenerated - when you select `Tools -> Create Desktop Entry` or when you update the IDE.


Now IntelliJ should see the variables. But not necessarily expose it to the program you run.
+ setting variables as visible https://stackoverflow.com/questions/45696203/intellij-idea-global-environment-variable-configuration?rq=1 (?) - the same might apply to other functionalities (e.g. Terminal) or specific runs.
