---
title: PASS setup
permalink: pass-setup
---

---
<!-- TOC -->

- [Tomb file for pass](#tomb-file-for-pass)
- [Initialize password-store](#initialize-password-store)
- [pass-split configuration](#pass-split-configuration)
    - [vault configuration](#vault-configuration)
    - [dom0 configuration](#dom0-configuration)
    - [template configuration](#template-configuration)
    - [joe-devq configuration](#joe-devq-configuration)
- [Usage](#usage)

<!-- /TOC -->

---

It's now time to configure `pass` the credentials-manager.

# Tomb file for pass

I want to force `pass` to store its pass-files in a tomb-file. I create the tomb-file by setting these global vars:

``` bash
IDENTITY="joe"
RECIPIENT="joe@foo.bar"
LABEL="cabal"
TOMBID="${IDENTITY}-${LABEL}"
SIZE=50
risks open gpg ${IDENTITY}
```

and following the [standard procedure](tomb-file-howto).

> Don't change `${TOMBID}` or `risks` won't work as expected.

# Initialize password-store

Once the tomb-file creation is over I open it with:

``` bash
risks open pass ${IDENTITY}
```

I initialize the password repository (`~/.password-store`):

``` bash
pass init ${RECIPIENT}
```

> Notice: `${RECIPIENT}` is the email used for the GPG configuration which in this case acts as GPG key identifier

I can now start using `pass` and save my website credentials (or any kind of note) with it.

> Check `man pass` for more details.

This is just a sample of how a pass-file looks like:

``` bash
123#stha36ea93-1
---
domain: www.reddit.com
username: joe@foo.bar
url: https://www.reddit.com

```

> Notice: The password is always saved in the first line of the file and it's standard for `pass`

# pass-split configuration

There is the option to configure some other AppVM to use `pass` in split mode, the same way as for GPG.

The pass-split project is hosted [here](https://github.com/Rudd-O/qubes-pass) and I need to download it.

So in _joe-devq_:

``` bash
git clone https://github.com/Rudd-O/qubes-pass.git
cd qubes-pass
```
Now I copy the files to the relevant machines:

These are for _vault_:

``` bash
qvm-copy etc/qubes-rpc/ruddo.PassRead # I select vault
qvm-copy etc/qubes-rpc/ruddo.PassManage # I select vault
```

and these are for _debian-10-dev_ (_joe-devq_ template):

``` bash
qvm-copy bin/*  # I select _debian-10-dev_ template
```

## vault configuration

``` bash
sudo mv ~/QubesIncoming/<git AppVM>/ruddo.* /etc/qubes-rpc/
```

## dom0 configuration

Dom0 is then configured to ask for confirmation both for read and write requests:

``` bash
sudo sh -c 'echo $anyvm $anyvm ask > /etc/qubes-rpc/policy/ruddo.PassManage'
sudo sh -c 'echo $anyvm $anyvm ask > /etc/qubes-rpc/policy/ruddo.PassRead'
```

Eventually, if I really trust _joe-devq_ and myself, I can authorize _joe-devq_ by default:

``` bash
sudo sh -c 'echo joe-devq vault allow > /etc/qubes-rpc/policy/ruddo.PassManage'
sudo sh -c 'echo $anyvm $anyvm ask >> /etc/qubes-rpc/policy/ruddo.PassManage'

sudo sh -c 'echo joe-devq vault allow > /etc/qubes-rpc/policy/ruddo.PassRead'
sudo sh -c 'echo $anyvm $anyvm ask >> /etc/qubes-rpc/policy/ruddo.PassRead'

```

## template configuration

In _debian-10-dev_

``` bash
sudo mv ~/QubesIncoming/<git AppVM>/qubes-pass-client /usr/bin/
sudo mv ~/QubesIncoming/<git AppVM>/qvm-pass /usr/bin/
sudo chmod +x /usr/bin/qubes-pass-client
sudo chmod +x /usr/bin/qvm-pass
sudo shutdown -P 0
```

## joe-devq configuration

I have hard time remembering all theses scripts so I create an alias which allows me to use `pass` instead of `qvm-pass` in _joe-devq_

``` bash
sudo sh -c 'echo "vault" > /rw/config/pass-split-domain'
echo 'alias pass="qvm-pass"' >> ~/.bash_aliases
sudo shutdown -P 0
```

# Usage

In order be able to use `pass` in _joe-devq_:

* I turn on _vault_
* attach the _hush partition_ from dom0 to _vault_ (`attach_hush_to`)
* I open a terminal in _vault_ and type:

``` bash
risks mount hush
risks open gpg ${IDENTITY}
risks open pass ${IDENTITY}
```

or
``` bash
risks mount hush
risks open identity ${IDENTITY} #this opens GPG, pass (and ssh) in one shot
```

Then I open a terminal in _joe-devq_ and I use `pass` without any difference from the usual behavior except for the fact that each time use it I have to clear the Qubes confirmation pop-up.