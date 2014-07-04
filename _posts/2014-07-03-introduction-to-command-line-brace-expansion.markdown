---
layout: post
title: "Introduction to Command-line Brace Expansion"
date: 2014-07-03 19:51
comments: true
categories: command-line
---

The command-line is a versatile interface that provides numerous tools for managing files and directories.
Due to its flexibility, renaming and copying files can be tedious and error-prone, especially when files are nested deep within a file system. Luckily we can solve this problem with a helpful feature called brace (or bracket) expansion.

Brace expansion, `{}`, is a method to create arbitrary strings. When combined with other commands, it can be a very powerful addition to a developer's toolbox.

## Copying Files

The first example we'll look at is for copying files. Let's start by creating an empty file called `a.txt` that we'll copy to a file called `b.txt`.

```
touch a.txt
```

Just to confirm we have a single file named `a.txt`, we can use `ls -lh`
and see the follow output:

```
-rw-r--r--  1 peterbrown  staff  0 Jul  3 19:04 a.txt
```

When copying files, the `cp` command requires two arguments (source and destination), and looks like this:

```
cp a.txt b.txt
```

The result from that command will look something like:

```
-rw-r--r--  1 peterbrown  staff     0B Jul  3 19:04 a.txt
-rw-r--r--  1 peterbrown  staff     0B Jul  3 20:18 b.txt
```

Now let's look at the same example, but this time using brace
expansion:

```
cp {a,b}.txt
```

Aside from the timestamp, the result from this command will be nearly identical to the one before:

```
-rw-r--r--  1 peterbrown  staff     0B Jul  3 19:04 a.txt
-rw-r--r--  1 peterbrown  staff     0B Jul  3 20:19 b.txt
```

While the results are identical, instead of specifying two separate file names, we have used curly braces to copy the file with a single argument. The first item in the braces, `a`, is the part of the original file name that we want to match, and the second item, `b`, is what we want to replace it with in the new file. Any text immediately surrounding the braces (in this case `.txt`) will be combined with the expansion.

## How Does it Work?

At first glance this syntax may seem foreign, so let's use the `echo` command to inspect what is happening behind the scenes:

```
echo {a,b}.txt
```

This results in the following output:

```
a.txt b.txt
```


In the output above, you can see that the comma-separated values within the braces are combined with the surrounding text, expanding it into two separate file names - exactly the input used previously to copy the `a.txt` file to `b.txt`.

If we combine the output from `echo` with `cp`, we have a command that is identical to the following:

```
cp a.txt b.txt
```

## Creating Backup Files

Brace expansion can be used with any commands that accept multiple arguments such as `cp`, `mv`, or even `git mv`. One use case for it is to quickly create backup files by adding an extension such as `.bak`.

Instead of having to type the same file name twice, and potentially spelling it wrong, we can use braces to add the extension:

```
mv a.txt{,.bak}
```

Which results in:

```
-rw-r--r--  1 peterbrown  staff     0B Jul  3 19:04 a.txt.bak
-rw-r--r--  1 peterbrown  staff     0B Jul  3 20:19 b.txt
```

Notice that in this example there is no value *before* the comma. This is because we are not replacing existing parts of the file name, and are just adding a new extension to it.

## Renaming Files in Git

Another common use is to rename files that are versioned with Git. Let's
create a quick Git repo and add our files to it:

```
git init
git add a.txt.bak b.txt
```

With `git status` we can see that both files have been staged:

```
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   a.txt.bak
	new file:   b.txt
```

Now we can use `git mv` with brace expansion to rename one of the files.
This time there is no value *after* the comma because we are removing part
of the file name.

```
git mv a.txt{.bak,}
```

Using `git status` again, we can see that `a.txt.bak` is now named `a.txt`.

```
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   a.txt
	new file:   b.txt
```

## Wrap Up

We've just barely scratched the surface of what is possible with brace expansions. As illustrated, this syntax can be combined with many of the common commands you use on a daily basis. I encourage you to explore them further and see if you can come up with a a clever use case on your own.
