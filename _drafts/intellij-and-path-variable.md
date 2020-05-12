---
layout: post
title:  "[EN] Environment variables in IntelliJ IDEA"
date:   2020-05-12 06:28:00 +0200
tags: [zsh, bash, intellij, env variables, linux, env variables]

----

Recently I encountered an issue with how IntelliJ reads environment variables. In one project, I was helping with, Maven build was using external `npm` command to build some frontend files. Unfortunately, when I started IntelliJ IDEA using a desktop shortcut (in Ubuntu) and ran Maven build inside the IDE it didn't see my `PATH` variable defined in configuration files (neither zsh nor bash).

## How to solve that?

It took me quite a lot of googling, 'stackoverflowing' and experiments to figure that out, so I hope it will help someone.

First of all, check if reading environment variables is enabled for the desired functionality. In this case, Maven builds, but the same applies to Terminal, specific build runs etc.

Open `File -> Settings -> Build, Execution, Deployments -> Build Tools -> Maven -> Runner -> Environment variables` and tick `Include system environment variables`. There you can also observe what variables are visible to IDEA.

{% include image.html
            img="/assets/intellijMavenEnvVariables.png"
            alt="IntelliJ example"
            caption="Include system environment variable in Maven build"
%}

If this didn't help it means that IDEA ignored your environment configuration files like it did in my case.

### Start IDE from a shell

Starting the IDE from a shell is one option, the easiest one. By doing that it will inherit variables visible in the current session. But it can be annoying.

Imagine that you already have a session running, which was started using `.desktop` file and you notice that your variables are not there. So you have to start it again, this time using the console. Annoying, isn't it? There must be a better way. And there is.

### Modify your configuration files

As it [turns out](https://youtrack.jetbrains.com/issue/IDEABKL-7589) IDEA starts using login shell. It means that during startup files like `.bashrc` or `.zshrc` are not read. Instead, you can put your configuration in `~/.profile` and re-login (or even better - restart your machine).

It's kind of surprising to me how it works, although it might be connected to how Gnome starts programs or how GDM starts environment, not necessarily IDEA startup scripts. It turns out that it does not read `~/.zprofile`, even though I use zsh. It also ignores `~/.bash_profile`, even though it should [take precedence](https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html) over `~/.profile` during login shell startup. Kind of a mess, isn't it?


Anyway, setting your variables in `~/.profile` works. But, what if you don't want to reconfigure all your startup scripts just to workaround one problem?

### Modify shortcut

You can either create some dummy `~/.profile` (or edit existing one) which will read your desired configuration file. Or you can modify `.desktop` shortcut `~/.local/share/applications/jetbrains-idea.desktop` and replace line

```bash
Exec="/home/amroz/Dev/ide/idea-IU-183.4284.148/bin/idea.sh" %f
```

with
```bash
Exec=zsh -c "/home/amroz/Dev/ide/idea-IU-183.4284.148/bin/idea.sh" %f
```

change `zsh` to `bash` if you don't use zsh.

From the manual:
```
-c Take  the  first  argument  as a command to execute, rather than reading commands from a script or standard input. If any further arguments are given, the first one is assigned to $0, rather than being used as a positional parameter.
```

In essence, it will run IDE using zsh and read `~/.zshenv`.

A word of warning here, though. Your changes might be overwritten if the `.desktop` file is regenerated (when you select `Tools -> Create Desktop Entry`) or when you update the IDE.
