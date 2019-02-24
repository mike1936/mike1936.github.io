---
layout: post
title: How to make a GPG-signing commit?
author: Mike
keyword: Security
tags: #security #GPG #commit #git
excerpt: To ensure the security of GIT commits while collaborating with others in a large project
postid: 5
---
## 1. Install GnuPG, generate the key with command
`gpg --gen-key`

A GPG key generally contains following infomations (something not related with the commit signing process is obmited, such as comments):

1). **Name**;

2). **Email**;

3). **Passphrase** -- !! KEEP IT SECRET !! , Passphrase is used for generating private key and verify your identity along with your private key, like a master-password of a database;

4). **Public-key series**, which includes:
- **Cipher text block** -- a plain text block, saves in a file with *.ASC extension, includes the most unabridged, complete information of the public key
- **Fingerprint** -- a string, "short version" of the cipher text block, used for identification
- **Key ID** -- a string, even "shorter version" of the fingerprint, mostly is the last 16bit of the fingerprint, for convenience, used in verification, like in this case

5). **Private-key series**, !! KEEP IT SECRET !!, 
- **Cipher text block** !! KEEP IT SECRET !! one can easily derive the public-key cipher given the private one but not vise versa (because of the hardness to do prime factorization of a large number, which is the basis of RSA algorithm).
- **Passphrase** !! KEEP IT SECRET !! as mentined before

#### Note:
It is possible to 'recover' the **Public key Fingerprint** and **Key ID** 'from' a **Private Cipher text block**, by importing the Private Cipher text block as a *.ASC file into a GPG license management tool such as 'Kleopatra', meaning that the **Private Cipher text block** 'kind of' contains all information of the whole GPG key. 

## 2. Get your Public-key Cipher text block
Use command `gpg --list-keys` in terminal to take a snapshot of your public key infomation.

The output should contains a tab named 'pub' with a long CAPPED string, that is your **Public-key fingerprint**.

Then `gpg --armor --export [Public-key fingerprint]` should get you the **Public-key Cipher text block** for next step.

## 3. Github Website setup
Go to https://github.com/settings/gpg/new, put your **Public-key Cipher text block** in and confirm.

## 4. git setup in terminal
``` sh
# Incase your GnuPG is not installed in a default folder, to makesure git can found the gpg executive, you need to set before use:
git config --global gpg.program [some_folder\GnuPG\bin\gpg.exe]
# Then set the following:
git config --global user.name [Name]
git config --global user.email [Email]
git config --global user.signingkey [Private-key Fingerprint / Key ID]
git config --global commit.gpgsign true
# (Optional):
git config -l # list all git configurations:
git config --list -- show-origin # list git configuration file path
```
## 5. Try commit and it should work
You might need a third party GPG license management software such as 'Kleopatra' to enable the passphrase input pop-up prompt for each commit.

Or you can simply ommit the passphrase (not secure) during GPG key generation.