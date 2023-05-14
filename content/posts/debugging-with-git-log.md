---
title: "Debugging with git log: A Useful Git Command for Tracking Changes"
date: 2023-05-14T00:00:00-07:00
draft: false
---

One of the most utilized git commands is `git log`. The main use of this command is to list the commit history of a branch. However, this command is many times more powerful with the ability to track down the exact location of a bug.

In this article, we will discuss flag options to use with `git log` to uncover it's true debugging potential.

## The git log -p flag

The first flag we will discuss is `git log -p`. When paired with a file name, 
```bash
git log -p <path-to-file>
```
this command will list all of the commits and accompanying changes of a file.

At times you may have a hunch that a bug resides within a specific file or group of files. This flag is great for revealing the history of the file in question. 

It works by display a commit hash and showing the difference between successive commits within a file. As shown below, we can see the exact changes performed under commit `788d8d`:

{{< image
src="/debugging-with-git-log/git-log-p-option.png"
width="90%"
height="90%"
alt="Screenshot of git log -p" >}}

Displaying the commit hash is an added bonus. We can use this commit hash in conjunction with

`git show <commit_hash>` or `git show --name-only <commit_hash>` 

to find all files touched under a particular commit. This is great as this gives us a clearer understanding as to why this file and associated files were changed.

This is very powerful command when trying to pinpoint the root cause of a bug. Start by obtain the commit hash. The commit hash can help you locate the ticket with the requirements that caused the code change. With the ticket and requirements in hand, a clearer picture will form as to why the code needed to be changed. This will hopefully making it simpler to implement a fix if need be.

## The git log -L flag

Another flag option that we have is `git log -L`. This flag is very similar to `git log -p` and works by retrieving more refined results within a file. It takes the form of 
```bash
git log -L <starting-line-number>,<ending-line-number>:<path-to-file>
``` 
and as you can tell by the parameters, this option targets a certain section of a file. 

Instead of retrieving all commits performed on a file, it returns only commits within the line segment specified. 

Use this command when you are certain that a bug resides within a function or set of functions. It will help you determine when these functions were last changed and why they were changed. With those details in hand, implementing the fix will be a much smoother process.

## Extra: gitk command

One last command that I would like to mention is `gitk`. It is great for those seeking a more visual friendly option. This commands works very similar to `git log -p`, when `gitk` is paired with a filename, `gitk <path-to-file>`. The added bonus is that `gitk` will launch an interactive GUI pop-up.

{{< image
src="/debugging-with-git-log/gitk-command.png"
width="90%"
height="90%"
alt="Screenshot of gitk" >}}

If you prefer a more interactive GUI, then this option may be for you. This command will also come in handy if you are after a much older commit, instead of scrolling with `git log -p`, you use the GUI to go directly to that commit. 

## Summary

`git log` is a very powerful command. It is great for showing our commit history. However, it has many more uses. 

In this post, we covered 2 flags that can assist you in your debugging journey: `git log -p` and `git log -L`. They are great tools to add to your toolbox. They can not only help you uncover where a bug is occurring but can also assist you in revealing why it was introduced in the first place.

Thanks for reading!