---
layout: post
title:  "Using Email on the Terminal with Mutt"
date:   2021-08-24 09:30:00 -0300
categories: mutt email linux
---

A great resource to learn how to use Mutt is the video:<br>
[Email on the terminal with mutt](https://www.youtube.com/watch?v=2jMInHnpNfQ) by Luke Smith

### Outlook Configurations

[https://support.microsoft.com/pt-br/office/configura%C3%A7%C3%B5es-pop-imap-e-smtp-para-outlook-com-d088b986-291d-42b8-9564-9c414e2aa040](https://support.microsoft.com/pt-br/office/configura%C3%A7%C3%B5es-pop-imap-e-smtp-para-outlook-com-d088b986-291d-42b8-9564-9c414e2aa040)

### My Configuration

`.config/mutt/muttrc`

```sh
set ssl_starttls=yes
set ssl_force_tls = yes

set sort = reverse-date-received
set editor = "vim"

# Sidebar
set sidebar_visible
set sidebar_format = "%B%?F? [%F]?%* %?N?%N/?%S"
set mail_check_stats

set imap_user = "your-email@hotmail.com"
set imap_pass = "#############"
set smtp_url = "smtp://$imap_user@smtp-mail.outlook.com:587"
set smtp_pass = $imap_pass

set from = $imap_user
set realname = "Your Name"

# Mailboxes
set folder = "imaps://outlook.office365.com:993"
set spoolfile = "+INBOX"
set record ="+Sent"
set trash = "+Trash"
set postponed="+Drafts"

mailboxes =INBOX =Sent =Trash =Drafts

source colors.muttrc

set date_format="%F"
set index_format="%4C %Z %d %-15.15L (%?l?%4l&%4c?) %s"

# Where to put the stuff
set mail_check = 100
set header_cache = "~/.config/mutt/cache/headers"
set message_cachedir = "~/.config/mutt/cache/bodies"
set certificate_file = "~/.config/mutt/certificates"

```

### Scroll text

>Hitting <return> in my Mutt instance will scroll the message one line at
>a time. The command mapped to it is called "next-line", so you can map
>it to anything you want in your .muttrc
>
>HTH,
>Tim Hammerquist

### Sort Order

From [How to make Mutt list messages in a descending order](https://bbs.archlinux.org/viewtopic.php?id=131257):

>I have recently switched to Mutt myself, and the manual is unfortunately not a good place to start. I have pieced together a nice config mainly by googling and stealing stuff from other people's configuration files.
>
>The setting you want is
>
>```sh
>set sort = reverse-date-received
>```
>
>That will show the most recent mails on top.
>
>If you want it threaded according to the most recent e-mail in a thread, this should work:
>
>```sh
>set sort=threads
>set sort_browser=reverse-date
>set sort_aux=last-date-received
>```
>
>Alternatively you might use
>
>sort_aux=reverse-last-date-received

### Select Multiple Messages

>Tag messages
>
>Mutt lets you "tag" multiple messages for action so that you can copy or delete them all at once with a single command. To use this feature, select each message with the t key command. Mutt will place an asterisk next to the message, indicating that it has been tagged. Once tagging is complete, use the ;c or ;d key shortcuts to copy or delete all the tagged messages simultaneously.

source: [https://www.techrepublic.com/article/10-helpful-tips-for-mutt-e-mail-client-power-users/](https://www.techrepublic.com/article/10-helpful-tips-for-mutt-e-mail-client-power-users/)

### Mutt Cheat Sheet

[![Mutt Cheat Sheet](/assets/images/mutt-cheat-sheet.png)](/assets/images/mutt-cheat-sheet.png)
Source: [http://sheet.shiar.nl/mutt](http://sheet.shiar.nl/mutt)

## Using GPG to encrypt your password

#### Create private and public key pairs

```sh
$ gpg --gen-key
gpg (GnuPG) 2.2.4; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Note: Use "gpg --full-generate-key" for a full featured key generation dialog.

GnuPG needs to construct a user ID to identify your key.

Real name: leandro
Email address: xyz@gmail.com
You selected this USER-ID:
    "le <xyz@gmail.com>"

Change (N)ame, (E)mail, or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 1U1U1U1U1U1U1U1U marked as ultimately trusted
gpg: directory '/home/leandro/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/leandro/.gnupg/openpgp-revocs.d/88888888658040234B8B61421U1U1U1U1U1U1U1U.rev'
public and secret key created and signed.

pub   rsa3072 2021-08-29 [SC] [expires: 2023-08-29]
      88888888658040234B8B61421U1U1U1U1U1U1U1U
uid                      le <xyz@gmail.com>
sub   rsa3072 2021-08-29 [E] [expires: 2023-08-29]
```

>sec => 'SECret key'<br>
>ssb => 'Secret SuBkey'<br>
>pub => 'PUBlic key'<br>
>sub => 'public SUBkey'

source: [What do 'ssb' and 'sec' mean in gpg's output?](https://superuser.com/questions/1371088/what-do-ssb-and-sec-mean-in-gpgs-output)

>You need to know the OpenPGP protocol to understand what this is about. IIRC,
>the GPH also explains this.
>
>pub = public key oacket<br>
>uid = user id packet<br>
>sub = public subkey packet<br>
>sec = secret key packet<br>
>sbb = secret subkey packet<br>
>sig = key signature

source: [https://dev.gnupg.org/T1563#122298](https://dev.gnupg.org/T1563#122298)


#### List keys

```sh
gpg --list-secret-keys --keyid-format=long
```

#### Print public key

```sh
$ gpg --armor --export 1U1U1U1U1U1U1U1U

-----BEGIN PGP PUBLIC KEY BLOCK-----

mQGNBGErvJ8BDACawEcAbXenFz4ZOa1/EHvNzWkveYAEpPW4H2rDahCVkeXF5VLo
q+rd6wsl+J7WQ/v93rlCa+i//cEK1Vy5hZ1uSZAr+u2eORSLIiNyQs5WBUEJ+eBt
...
SuNmAnu/MqvYVsQp9ftkbOfiJWipduPIfu/ry3R02Zwdgsztz5DMxZMCATFF8IcB
HHc++EFE2/I9K2zWiQ4O+ot8WSGEgGDomA==
=Wp9y
-----END PGP PUBLIC KEY BLOCK-----
```

#### Encrypting a file

```sh
gpg --output imap_pass.gpg --encrypt --recipient leandro tmp
```

where `recipient` can be: key, name or email.

#### Decrypting a file

The command below will ask you for the passphrase.

```sh
gpg --decrypt imap_pass.gpg
```

#### set imap_pass for use the encrypted file

```sh
set imap_pass="`gpg --batch -q --decrypt ~/.config/mutt/imap_pass.gpg`"
```

## Important Commands

### Quit

q

### Undo

u

### Cancel a Command in Mutt

>To cancel a command in the Mutt email client, press control and "g" at the same time.
>
>`ctrl+g`
>
>For years, I had been using ctrl+c, which cancels the command but also asks if you want to close mutt -- so you had to also type "n" and enter.

source: [https://magnatecha.com/cancel-a-command-in-mutt/](https://magnatecha.com/cancel-a-command-in-mutt/)

### Select and delete multiple messages

`t` is used to "tag" (select) messages. Then you can type `;d` to delete them.

### Move message to a different mailbox

`s`

### Change mailbox

`c`

## References

[Mutt cheat sheet](http://sheet.shiar.nl/mutt)

[Email clients info for Linux](https://www.kernel.org/doc/html/latest/process/email-clients.html)

[https://neomutt.org/guide/configuration.html](https://neomutt.org/guide/configuration.html)

[https://gitlab.com/muttmua/mutt/-/wikis/UseCases/Gmail](https://gitlab.com/muttmua/mutt/-/wikis/UseCases/Gmail)

[Hotmail template config for mutt, just insert your mail account and password to imap_user and imap_pass, if 2-factor is enable, generate an app password.](https://gist.github.com/yangxuan8282/a18d757429c2e3a89699325045c742b3)

[Email on the terminal with mutt](https://www.youtube.com/watch?v=2jMInHnpNfQ)

[https://support.microsoft.com/pt-br/office/configura%C3%A7%C3%B5es-pop-imap-e-smtp-para-outlook-com-d088b986-291d-42b8-9564-9c414e2aa040](https://support.microsoft.com/pt-br/office/configura%C3%A7%C3%B5es-pop-imap-e-smtp-para-outlook-com-d088b986-291d-42b8-9564-9c414e2aa040)

[https://unix.stackexchange.com/questions/43250/how-can-i-make-mutt-show-date-field-of-mail-on-the-index-screen](https://unix.stackexchange.com/questions/43250/how-can-i-make-mutt-show-date-field-of-mail-on-the-index-screen)

[colors.muttrc](https://gist.githubusercontent.com/LukeSmithxyz/de94948264649a9264193e96f5610c44/raw/d274199d3ed1bcded2039afe33a771643451a9d5/colors.muttrc)

[When reading an e-mail in Mutt, how can I scroll line by line?](https://comp.mail.mutt.narkive.com/cCi3OuYZ/when-reading-an-e-mail-in-mutt-how-can-i-scroll-line-by-line)

[mutt: save message to specific folder](https://unix.stackexchange.com/questions/102829/mutt-save-message-to-specific-folder/104138)

[10 helpful tips for Mutt e-mail client power users](https://www.techrepublic.com/article/10-helpful-tips-for-mutt-e-mail-client-power-users/)

[The Mutt Cheat Sheet](https://www.ucolick.org/~lharden/muttchart.html)

[Generating a new GPG key](https://docs.github.com/en/github/authenticating-to-github/managing-commit-signature-verification/generating-a-new-gpg-key)

[GnuPG Basics Explained with Linux GPG Command Examples](https://www.thegeekstuff.com/2012/10/gnupg-basics/)

[Encrypting and decrypting documents](https://www.gnupg.org/gph/en/manual/x110.html)

[How to use GPG to encrypt stuff](https://yanhan.github.io/posts/2017-09-27-how-to-use-gpg-to-encrypt-stuff/)
