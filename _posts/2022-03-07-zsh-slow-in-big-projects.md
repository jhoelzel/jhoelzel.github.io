---
layout: post
title: ZSH slow response times
subtitle: or how big repositories slow down your theme
categories: cli
tags: [zsh, tips, cli, git, theme]
---

Today I found myself with a slow unresponsive zsh console session in certain folders and I spent some time figuring out why.
What happen was that every input to my console took like 20 seconds to finish and it was having an impact on my work.

It was driving me mad, I rebuild my dev container, rebooted to no avail until I realized: ** the size of the Repo has an impact on my terminal? **
Turns out that is exactly what happened. And here is why:

## Themes and Git Info

There are two central git functions in in lib/git.zsh:

```
    # Outputs current branch info in prompt format
    git_prompt_info()

    # Checks if working tree is dirty
    parse_git_dirty()
```

Which run on every command return in order to show you the nice little graphics that indicate that your current branch has unstaged commits.

Luckily each method has a git config switch to disable it:

```
    oh-my-zsh.hide-status to disable git_prompt_info()
    oh-my-zsh.hide-dirty to disable parse_git_dirty()
```

Some themes create their own git queries and sometimes ignore these flags but in general they can be deactivated using the following flags:

``` Console
$ git config --add oh-my-zsh.hide-status 1
$ git config --add oh-my-zsh.hide-dirty 1
```

## Explanation

In my case I was working on a rather large monorepo and the problem manifested itself quickly after that. Checking if my working tree is dirty took a considerable amount of time and doing so on every command return even more so.
With Visual Studio code, dirty tracking is already set up for me in a separate extension anyway so I would not care if a dirty check is performed every time.

### Sources

<https://github.com/ohmyzsh/ohmyzsh/blob/master/lib/git.zsh>
<https://stackoverflow.com/questions/12765344/oh-my-zsh-slow-but-only-for-certain-git-repo>
<https://blog.carbonfive.com/writing-zsh-themes-a-quickref/>
