---
layout: post
title:  "Learning how to use git send-email"
date:   2021-08-22 08:10:00 -0300
categories: git send-email 
---

# git send-email

Why learn how to use use `git send-email`?

From: [https://git-send-email.io](https://git-send-email.io)

> email + git = <3
>
>Git ships with built-in tools for collaborating over email. With this guide, you'll be contributing to email-driven projects like the Linux kernel, PostgreSQL, or even git itself in no time. 

This is a great interactive site that teaches you how to use `git send-email`


I'll follow the site instructions on Ubuntu:

## Installation

>The git-email package includes the git email tools. Run this to install it:

`sudo apt install git git-email`

## Configuration

You can have a global configuration and per git project configuration.

The global configuration is done in the file `~/.gitconfig`, eg for Outlook:

```git
[user]
    email = your@hotmail.com
    name = Learning Sendemail
[sendemail]
    smtpserver = outlook.office365.com
    smtpencryption = tls
    smtpuser = your@hotmail.com
```

For Gmail:

```git
[sendemail]
    smtpServer = smtp.gmail.com
    smtpServerPort = 587
    smtpencryption = tls
    smtpuser = xyz@gmail.com
```

If you have Two-Factor Authentitication (2FA) for your email (which I hope you do =)), then
you need to create an `app password` and use it to send the email.

[https://support.google.com/accounts/answer/185833?p=InvalidSecondFactor](https://support.google.com/accounts/answer/185833?p=InvalidSecondFactor)

## Sending the last commit

`git send-email --to="destination@hotmail.com" HEAD^`


## Sending version 2

From:
[https://git-send-email.io/#step-4](https://git-send-email.io/#step-4)

Change some file and update the commit: `git commit -a --amend`

Then send the patch:

`git send-email --annotate -v2 --to="destination@hotmail.com" HEAD^`

>Note that we also specified the "--annotate" flag. This is going to open the email in our editor before sending it out, so we can make any changes. We're going to add some "timely commentary". Look for the "---" and add a short summary of the differences since the first patch on the next line. It should look something like this:
>
>```patch
>Subject: [PATCH v2] Demonstrate that I can use git send-email
>
>---
>This fixes the issues raised from the first patch.
>
>your-name | 1 +
>1 file changed, 1 insertion(+)
>```
>
>This text gives the maintainers some extra context about your patch, but doesn't make it into the final git log.
>
> Source: [https://git-send-email.io/#step-4](https://git-send-email.io/#step-4)

## Using --annotate every time

`git config --global sendemail.annotate yes`

## Signing your commits patches

`git config --global format.signOff yes`

ps! This does not add `Signed-off-by` to your local commits. It's added
only when the patch is sent.

## Sending the last two commits

`git send-email --to="xyz@hotmail.com" HEAD~2`

It's nice that it automatically create a patch series, eg:

1. [PATCH 1/2] First commit header
2. [PATCH 2/2] Second commit header


## Other nice tips

### Replying on the same thread in a mailing list using git send-email

>Use Message-id instead of the subject in the --in-reply-to option.
>
>`--in-reply-to=<Message-id>`
>
>You can find the Message-Id value in the header of the message you want to reply to.
>
> [https://stackoverflow.com/questions/26404327/replying-on-the-same-thread-in-a-mailing-list-using-git-send-email](https://stackoverflow.com/questions/26404327/replying-on-the-same-thread-in-a-mailing-list-using-git-send-email)

### Message IDs

>>The git email client seems to have support for message IDs.
>
>You should never have to directly work with message IDs if you're using git-send-email and letting your mail client handle replies.
>
> [https://news.ycombinator.com/item?id=24935979](https://news.ycombinator.com/item?id=24935979)

### Save your app specific password

Gabriel Staples has a great [answer](https://stackoverflow.com/a/68238913/339561) for the question [How to configure and use git send-email to work with gmail to email patches to developers](https://stackoverflow.com/questions/68238912/how-to-configure-and-use-git-send-email-to-work-with-gmail-to-email-patches-to).<br>
One of the things he explains is how to use `git credentials`:

>Since you added the [credential] helper = store entry to your ~/.gitconfig file above, git will now automatically store your app-specific password for future use in a new file called ~/.git-credentials. If you open that file, you'll see a URL string with your password embedded in it in plain text. It will look like this:
>
>smtp://EMAIL%40gmail.com:16-DIGIT-PASSWORD@smtp.gmail.com%3a587


# References

[Interactive git send-email tutorial](https://git-send-email.io)<br>
[Git send-email using Gmail ](https://gist.github.com/jasonkarns/4354421)<br>
[Git commit documentation](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt--a)<br>
[Git send-email documentation](https://git-scm.com/docs/git-send-email#Documentation/git-send-email.txt---in-reply-toltidentifiergt)

