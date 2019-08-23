    Title: Setting up Magit Forge with Github Enterprise server
    Date: 2019-08-23T16:14:14
    Tags: emacs, git


I have attempted a number of times to create a pull request from the comfort of
Emacs and Magit, with no success yet. The main problem is that I use most of the
time my company's GitHub Enterprise instance, not the public service called
`github.com`. The GitHub service is so ubiquitous that many people forget there
are other installations, not seen on the public Internet, where big corporations
and hip companies keep their precious, precious code.

This week I finally made progress in setting up the connection.

<!-- more -->

You need to customize the variable `forge-alist` to include your company's GHE
as a new "forge". You need to add there a four-element tuple that describes how
Magit Forge should talk to this new Github, and also how it should be identified
to the user. It is easier to see how the fields are intended if you use a
Customize buffer, but in the end, you need to do the equivalent of evaluating
this expression:

```emacs-lisp
(push '("ghe.company.net" "ghe.company.net/api/v3"
        "ghe.company.net" forge-github-repository)
        forge-alist)
```

where it is assumed that your company's sysadmins have deployed the Github
service under a DNS name like `ghe.company.net`. Of course, this will be
different for each deployment, and it could be called
`veryprivategithub.anotherdomainweown.com`. You need to figure out that bit.

Another thing that was tricky was that the next thing to do, on a repository
that has been cloned from a remote with this name, is to do a `forge-pull`. This
will try to contact the GHE instance and download data like open issues.

In order to connect to the forge, Magit Forge needs to know which credentials to
use. There is a wizard, for the first `forge-pull`, where you just need to give
your user and password (!) and it will create the right token for you using the
GHE API (and hopefully, immediately forget your credentials!).

If you have trust issues, or are a hacker that needs to do everything by
themselves, you can do the following instead:

1. Change the order of sources in the variable `auth-sources` and set
   `~/authinfo.gpg` **higher** than `~/authinfo`. This means that the
   **encrypted file** will be used in preference to a **plaintext file** in
   order to store the token that allows Magit to connect to GHE.
   
2. Open `~/.authinfo.gpg`. You may need to enter your passphrase for that file.
   You may need to **create** that file in the first place. You may need to
   **create a GPG private/public keypair** in order to encrypt files at all.
   Please refer to the GPG documentation for that. Ah! On top of that, when you
   try to open `~/.authinfo.gpg`, and you are using GUI Emacs version 26, you
   may need to `(setq epa-pinentry-mode 'loopback)`, otherwise it will **not**
   prompt you to enter your passphrase and just throw a decryption error. Go and
   do all of that, I will be waiting here.
   
3. Ready? OK, now create a token on the Github Enterprise API for your user account.

4. You should write that token into `~/.authinfo.gpg` in this format:

```
# -*- epa-file-encrypt-to: ("youruser@company.net") -*-
machine ghe.company.net/api/v3 login youruser^forge password <and the token that you just generated>
```
