---
layout: post
title: dotfiles Guide
categories: dotfiles
---

On Unix based systems dotfiles are configuration files that typically live in a users home directory `~/`. They start with a period. Some people spend countless hours pouring over their configurations. A problem with so much effort is can you use them on other machines? What if your computer crashes? What if I break an app with a configuration change?

Having organized dotfiles solves those problems and more. Also, as I organize and really dive into my dotfiles I see how much better I can be with tweaking systems to match my own desired workflows.

I recently dove into setting up [my dotfiles] and have posted them to GitHub.

### What are they?
Look in your home directory `~/` and you'll see files like `.gitconfig`, `.vimrc` and `.zshrc`. Those are your dotfiles. `.gitconfig` manages my git settings, `.vimrc` is my vim configuration, and `.zshrc` is my zsh shell profile.

### How do I use and organize them?
You just edit them... carefully. The problem is they are all over the place and within the home folder are quite a few generated dotfiles that you'll never touch. The best thing to do is get them all into a single folder. Then use git to manage versions of them. The problem is you can't move these files and reasonably expect the programs that rely on them to work. Unix solves this problem with a command line utility called `link`, more commonly referred to as `ln`. Specifically, we will use ln's symbolic link feature.

### How do I set them up?

First create the folder and initialize a git repo.

``` bash
> cd ~/
> mkdir .dotfiles
> cd .dotfiles
> git init
```

Next you'll move your dotfiles to the new folder.

``` bash
> cd ~/
> mv .gitconfig .dotfiles/gitconfig
> mv .vimrc .dotfiles/vimrc
> mv .zshrc .dotfiles/zshrc
```

Next, we'll replace the moved files with a symbolic link that performs any operations on the symbolic link to the actual file inside your ``.dotfiles` directory. For example any read or write operation on `~/.gitconfig` is done to `~/.dotfiles/gitconfig`.

``` bash
> ln -s .dotfiles/gitconfig .gitconfig
> ln -s .dotfiles/vimrc .vimrc
> ln -s .dotfiles/zshrc .zshrc
```

Don't forget to make your first commit.

``` bash
> cd .dotfiles
> git add .
> git commit -m "Initial dotfiles commit"
```

That's it. You now have a `.dotfiles` directory with your first few dotfiles being managed.

For a simple to use set of utilities to handle this whole process checkout the [rcm] suite.

[my dotfiles]: (http://github.com/wassimk/dotfiles)
[rcm]: https://github.com/thoughtbot/rcm
