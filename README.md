# ansible-role-passwordstore
Install and configure pass along with gpg2 on a workstation

# Manually generating GPG keys

> SOURCE: https://gist.github.com/flbuddymooreiv/a4f24da7e0c3552942ff

The following shell transcript shows how to:
  * Create a GPG key
  * Create a pass database
  * Add git support to the pass database
  * Create a remote git repository
  * Push the pass database to the remote git repository
  * Fetch and display your passwords from another host

It is assumed that the `pass` package has been installed on both the first and second computers.

## Create a GPG key
<pre>
<b>user@host:~$ gpg --gen-key</b>
gpg (GnuPG) 1.4.18; Copyright (C) 2014 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
<b>Your selection? 1</b>
RSA keys may be between 1024 and 4096 bits long.
<b>What keysize do you want? (2048) 4096</b>
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      &lt;n&gt;  = key expires in n days
      &lt;n&gt;w = key expires in n weeks
      &lt;n&gt;m = key expires in n months
      &lt;n&gt;y = key expires in n years
<b>Key is valid for? (0) </b>
Key does not expire at all
<b>Is this correct? (y/N) y</b>

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) &lt;heinrichh@duesseldorf.de&gt;"

<b>Real name: First Middle Last Suffix</b>
<b>Email address: first.last@host.tld</b>
Comment:
You selected this USER-ID:
    "First Middle Last Suffix &lt;first.last@host.tld&gt;"

<b>Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O</b>
You need a Passphrase to protect your secret key.

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 172 more bytes)
............+++++

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 198 more bytes)
..............+++++
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 250 more bytes)
..........+++++

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 249 more bytes)
......+++++
gpg: key 68214821 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
pub   4096R/68214821 2015-06-24
      Key fingerprint = A5C2 96E8 AC41 0889 60D9  2D1F 0F6D B722 6821 4821
uid                  First Middle Last Suffix &lt;first.last@host.tld&gt;
sub   4096R/36A6F06D 2015-06-24
</pre>

## Create a pass database
<pre>
<b>user@host:~$ gpg --list-keys</b>
/home/user/.gnupg/pubring.gpg
------------------------------
pub   4096R/68214821 2015-06-24
uid                  First Middle Last Suffix &lt;first.last@host.tld&gt;
sub   4096R/36A6F06D 2015-06-24

<b>user@host:~$ pass init 68214821</b>
mkdir: created directory ‘/home/user/.password-store/’
Password store initialized for 68214821
</pre>

## Add git support to the pass database
<pre>
<b>user@host:~$ pass git init</b>
Initialized empty Git repository in /home/user/.password-store/.git/
[master (root-commit) c343a0c] Add current contents of password store.
 1 file changed, 1 insertion(+)
 create mode 100644 .gpg-id
[master edaf464] Configure git repository for gpg file diff.
 1 file changed, 1 insertion(+)
 create mode 100644 .gitattributes

<b>user@host:~$ pass generate serviceprovider/account.name@service.tld 21</b>
mkdir: created directory ‘/home/user/.password-store/gmail’
[master e6a1974] Add generated password for serviceprovider/account.name@service.tld.
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 serviceprovider/account.name@service.tld.gpg
The generated password for serviceprovider/account.name@service.tld is:
;J&E_2A55&lt;%=&lt;KxoEDZuL
</pre>

## Create a remote git repository
<pre>
<b>user@host:~$ ssh user@gitrepo.org -p gitport \
    "git init --bare /path/to/git/user-password/"</b>
<b>user@gitrepo.org's password: </b>
Initialized empty Git repository in /path/to/version_systems/git/user-password/
</pre>

## Push the pass database to the remote git repository
<pre>
<b>user@host:~$ pass git remote add origin \
    ssh://user@gitrepo.org:gitport/path/to/git/user-password/</b>

<b>user@host:~$ pass git push -u --all</b>
<b>user@gitrepo.org's password: </b>
Counting objects: 10, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (10/10), 1.38 KiB | 0 bytes/s, done.
Total 10 (delta 1), reused 0 (delta 0)
To ssh://user@gitrepo.org:gitport/path/to/git/user-password/
 * [new branch]      master -&gt; master
Branch master set up to track remote branch master from origin.
</pre>

## Fetch and display passwords from another computer
It is assumed here that your GPG key has been migrated to the second computer.

<pre>
<b>user@host:~$ gpg --list-keys</b>
/home/user/.gnupg/pubring.gpg
------------------------------
pub   4096R/68214821 2015-06-24
uid                  First Middle Last Suffix &lt;first.last@host.tld&gt;
sub   4096R/36A6F06D 2015-06-24

<b>user@host:~$ pass init 68214821</b>
mkdir: created directory ‘/home/user/.password-store/’
Password store initialized for 68214821

<b>user@host:~$ pass git init</b>
Initialized empty Git repository in /home/user/.password-store/.git/
[master (root-commit) c343a0c] Add current contents of password store.
 1 file changed, 1 insertion(+)
 create mode 100644 .gpg-id
[master edaf464] Configure git repository for gpg file diff.
 1 file changed, 1 insertion(+)
 create mode 100644 .gitattributes

<b>user@host:~$ pass git remote add origin \
    ssh://user@gitrepo.org:gitport/path/to/git/user-password/</b>

<b>user@host:~$ git reset origin/master</b>

<b>user@host:~$ pass git fetch</b>
<b>user@gitrepo.org's password: </b>
warning: no common commits
remote: Counting objects: 10, done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 10 (delta 1), reused 0 (delta 0)
Unpacking objects: 100% (10/10), done.
From ssh://user@gitrepo.org:gitport/path/to/git/user-password/
 * [new branch]      master     -&gt; origin/master

<b>user@host:~$ pass git rebase origin/master</b>
First, rewinding head to replay your work on top of it...

<b>user@host:~$ pass show serviceprovider/account.name@service.tld</b>
gpg: WARNING: The GNOME keyring manager hijacked the GnuPG agent.
gpg: WARNING: GnuPG will not work properly - please configure that tool to not interfere with the GnuPG system!
;J&E_2A55&lt;%=&lt;KxoEDZuL
</pre>
