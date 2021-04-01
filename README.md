# Datalab SSH Policy

NOTE: (2021-03-24) this is a draft of our new policy, and is not enforced yet,
but you should go ahead and create keys in advance of the new policy coming
into effect. Policy should start to be the only way sometime in April 2021.

We use SSH as primary access to all our Datalab infrastructure and as well as
parts of the OpenSAFELY backend infrastructure. We also use it for Github
authentication for cloning and pushing. 

Thus, keeping our SSH keys and usage secure is very important. This document
describes our policy for SSH key management.


## The Key

You should have a dedicated Datalab SSH key for use with Datalab systems, and
not with other systems. This should be a modern ed25519 key, protected by
a strong passphrase, and have your Datalab email address and creation date in
the key's comment.

To generate such a key, you can run this in a terminal:

    ssh-keygen -t ed25519 -C "YOUREMAIL@thedatalab.org:$(date --iso-8601)" ~/.ssh/datalab_ed25519

Note: you MUST enter a strong password when prompted.


## Add to your Github Account

We use Github as a key distribution mechanism. You should add the contents of
your public key file (e.g. `~/.ssh/datalab_ed25519.pub`) to your Github
account:

[https://github.com/settings/keys](https://github.com/settings/keys)

Github doesn't preserve the key comment, so we recommend titling the key with
the comment, e.g.

     YOURRMAIL@thedatalab.org:202X-XX-XX


## Confirm your Key

Clone this repo and run the following to a ensure your new keys fingerprint is
added:

    ./add.py YOUR_GITHUB_USERNAME ~/.ssh/datalab_ed25519

This is should ensure your user has a correct entry with key fingerprint in the
`passwd` file, marking this key as your current Datalab key.

You need to add and commit this, but you MUST sign it with your registered
Github GPG key, or else the commit will be blocked.

Once this is reviewed and merged, after a short time your account and key
should be available to log in with on all our servers.

## Sudo Access and Passwords

By default, all of the tech team at the Datalab have sudo access on all our
infra. However, you do need a password set to use sudo.

For each server, the first time you log in via SSH, you will need to set
a password. It should be a strong password, but bear in mind you will need to
type it in by hand, most likley. It makes sense to reuse the same password
across multiple servers, but it's not required.

If you ever lose or forget your password, you can as one of the team to run the
following:

    sudo passwd -de YOUR_GITHUB_USERNAME

This will delete your password and force a reset next time you SSH in.

## Local SSH Config

You can use your local SSH config to ensure you use the right username and key when
accessing Datalab systems with the following snippet in `~/.ssh/config`


```
Host *.ebmdatalab.net *.opensafely.org
    User YOUR_GITHUB_USERNAME
    IdentityFile ~/.ssh/datalab_ed25519
```

This should allow you to just do `ssh somehost.embdatalab.net` and it Just Works.


## Key Compromise and Rotation

If your key is ever compromised (e.g. your computer is stolen or hacked), you
should inform Simon or Tom W ASAP, and immediately remove the public key from
your Github account, and also the fingerprint from this repo. Either of these
actions will cause the key to be removed from our various systems within
a short window. You can then add a new key as above.


You are encouraged to rotate your keys at least once every 12 months, ideally
more regularly.  You can rotate your key by generating it as above, and adding
it to Github and updating the fingerprint in this repo as described. After
a short while, your old key will be removed from our systems and your new key added.
You can then remove the old key from your github account.


## SSH Agents and Forwarding

Local SSH agents are encouraged, but you should *not* use agent forwarding by
default, as this [presents a significant
risk](https://smallstep.com/blog/ssh-agent-explained/#agent-forwarding-comes-with-a-risk).

You may occasionally need to use your SSH key on a remote server, e.g. to
commit and push changes made to repos on those remote machines.

To do so safely, you should temporarily re-connect with agent forwarding
enabled (`ssh -A ...`), then exit and go back to regular non-forwarded SSH
connection afterwards.


## Tests


We use docker to simulate a system to run on in test.

To run all tests:

    make test

To run specific test:

    make tests/basic.sh

To run test and drop into shell after running:

    make tests/basic.sh DEBUG=1
