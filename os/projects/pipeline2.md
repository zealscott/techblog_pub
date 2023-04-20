<p align="center">
  <b>接上一篇，本篇主要介绍字符串处理和命令的解析。</b>
</p>

# 命令解析
完成一个命令的解析，最重要的步骤就是字符串的解析。我们如何对拿到的字符串进行分解呢？笔者的思路如下：



1. 使用**fgets()**等函数将输入的命令存放在缓存区中。
2. 对其用空格对其进行分割（使用**strtok**等字符串处理函数），解析出特殊命令符（重定向">"，管道"|"，后台程序"&"等）
3. 识别出特殊命令：例如**history**，**回车**，**exit**，**cd**等，这些命令不能使用**exceve**函数进行解析和运行，需要单独处理。
4. 如果字符串中有特殊命令符，则需要对命令两边分别进行操作。

## 分割字符串
```C
/*
  Parse the command line with space and get the argv array
*/
void parseline() {
    initStruct(&CommandInfo);

    buf[strlen(buf) - 1] = ' '; /*Replace trailing '\n' with space*/

    /*split buf with space*/
    char* token = strtok(buf, " ");
    while (token) {
        CommandInfo.argv[CommandInfo.argc++] = token;
        token = strtok(NULL, " ");
    }
    /*set the last command with NULL*/
    CommandInfo.argv[CommandInfo.argc] = NULL;

    /*empty command line*/
    if (CommandInfo.argc == 0)
        return;

    /*indicate whether its background Command*/
    CommandInfo.background = (*(CommandInfo.argv[CommandInfo.argc - 1]) == '&');
    if (CommandInfo.background)
        CommandInfo.argv[--CommandInfo.argc] = NULL;
    return;
}
```


## 特殊命令处理
针对空格、history、cd等特殊命令，可以先做预处理。
```C
/*if return 1, ignore the command*/
int IgnoreCommand() {       
    /*if no command,continue*/
    if (CommandInfo.argc < 1)
        return 1;

    /*exit command*/
    if (strcmp(CommandInfo.argv[0], "exit") == 0)
        exit(0);

    /*history command*/
    if (strcmp(CommandInfo.argv[0], "history") == 0) {
        if (CommandInfo.argc == 1)
            /*print all the history*/
            PrintCommand(-1);
        else {
            PrintCommand(atoi(CommandInfo.argv[1])); /*convert string to int*/
        }
        return 1;
    }

    /*cd command to change directory*/
    if (strcmp(CommandInfo.argv[0], "cd") == 0) {
        if (CommandInfo.argc > 1) {
            if (chdir(CommandInfo.argv[1]) == -1) {
                printf("error directory!\n");
            }
        }
        return 1;
    }

    /*wrong command*/
    if (strcmp(CommandInfo.argv[CommandInfo.argc - 1], "<") == 0 ||
        strcmp(CommandInfo.argv[CommandInfo.argc - 1], ">") == 0 ||
        strcmp(CommandInfo.argv[CommandInfo.argc - 1], "|") == 0) {
        printf("Error:command error\n");
        return 1;
    }

    return 0;
}
```

## 解析命令操作符
对于“>”，“<”,“>>”操作符，不需要进行管道操作，因此直接先读取文件名。
```C
int ReviseCommand() {
    /*
    if the command is empty or exit or cd or history, should ignore the command;
    */
    if (IgnoreCommand())
        return -1;

    int i, override = 0;

    /*search the command with special charactors,and store the file and type*/
    for (i = 0; i < CommandInfo.argc; i++) {
        if (strcmp(CommandInfo.argv[i], "<") == 0) {
            CommandInfo.argv[i] = NULL;
            CommandInfo.file = CommandInfo.argv[i + 1];
            CommandInfo.type[CommandInfo.index++] = IN_REDIRECT;
            override = 1;
 
        } else if (strcmp(CommandInfo.argv[i], ">") == 0) {
            /* if > is not the first command, should not set the file */
            CommandInfo.argv[i] = NULL;
            if (!override)
                CommandInfo.file = CommandInfo.argv[i + 1];
            CommandInfo.type[CommandInfo.index++] = OUT_REDIRECT;
            break;
 
        } else if (strcmp(CommandInfo.argv[i], ">>") == 0) {
            CommandInfo.argv[i] = NULL;
            if (!override)
                CommandInfo.file = CommandInfo.argv[i + 1];
            CommandInfo.type[CommandInfo.index++] = OUT_ADD;
            break;
 
        }
        /*multi - PIPE*/
        else if (strcmp(CommandInfo.argv[i], "|") == 0) {
            CommandInfo.type[CommandInfo.index++] = PIPE;
            CommandInfo.argv[i] = NULL;
        }
    }
    return 1;
}
```

# 命令主题框架
我们首先使用**parseline()**对得到的命令按照空格进行解析，之后再使用**ReviseCommand()**提取关键命令字符，识别回车键等，最后再对进程进行**fork()**，子进程（ChildCommand）执行命令，父进程根据是否有“&”选择等待子进程结束或者继续执行。
```C
void command() {
    pid_t pid;
    int indicator = 0;

    parseline();

    /*re-edit command  and get the file*/
    indicator = ReviseCommand();

    if (indicator == -1)
        return;

    pid = fork();
    if (!pid) {
        /*the background process should not be
        disturbed by CTRL+C and CTRL+\*/
        /*sigaction(SIGINT, SIG_IGN, NULL);
        sigaction(SIGQUIT, SIG_IGN, NULL);*/
        ChildCommand();
        exit(0);
    } else {
        if (!CommandInfo.background)
            waitpid(pid, NULL, 0);
        else {
            /*if background process, the father should ignore the signal
               let init to reap it */
            printf("there is a background process\n");
        }
    }
    return;
}
```

# 子进程命令框架
对于fork出来的子进程，如果只有重定向这种简单的命令，我们通过解析到的字符串和文件名就可以直接进行操作，如果涉及到多个管道的操作，那就要小心了。
```C
void ChildCommand() {
    int fd;
    switch (CommandInfo.type[0]) {
        case NORMAL:
            Execvp(CommandInfo.argv[0], CommandInfo.argv);
            break;

        case IN_REDIRECT: /* < command*/
            fd = open(CommandInfo.file, O_RDONLY);
            if (fd == -1) {
                printf("Error: wrong input!\n");
                break;
            }
            dup2(fd, STDIN_FILENO);

            if (CommandInfo.type[1] == PIPE) {
                EditInfo();
                pipe_command();
            }
            Execvp(CommandInfo.argv[0], CommandInfo.argv);
            break;

        case OUT_REDIRECT: /* > command*/
            fd = open(CommandInfo.file, O_WRONLY | O_CREAT | O_TRUNC, 0666);
            dup2(fd, STDOUT_FILENO);
            Execvp(CommandInfo.argv[0], CommandInfo.argv);
            break;

        case OUT_ADD: /* >> command*/
            fd = open(CommandInfo.file, O_RDWR | O_APPEND, 0666);
            dup2(fd, STDOUT_FILENO);
            Execvp(CommandInfo.argv[0], CommandInfo.argv);
            break;

        case PIPE: /* | command*/
            pipe_command();
            break;
    }
}
```

这样，除了多管道以外的其他命令和要求我们基本上都实现了，管道的操作略微复杂，我专门写一篇来增强理解。


# 参考资料：
1. [Linux shell的实现][1]
2. Operating System:Design and Implementation,Third Edition 
3. Computer Systems: A Programmer's Perspective, 3/E

[1]: http://blog.csdn.net/tankai19880619/article/details/49678565