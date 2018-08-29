# Tezos Ledger Applications

## Overview

Whether you're baking or just trading XTZ, you want to store your keys securely.
This is what a "hardware wallet" like the
[Ledger Nano S](https://www.ledgerwallet.com/products/ledger-nano-s) is for.
Your private keys never leave the device, and it performs the signing
operations. To use a Ledger Nano S with [Tezos](https://www.tezos.com/), you need to
load Tezos-specific software onto it.

The term "hardware wallet" can refer to several devices that store your private
keys in a secure way. The term "wallet" refers to the fact that it stores your
"money" -- in the case of Tezos, it stores your tez. Remember, storing your
tokens means storing the private keys that control your tokens. But the wallet
also has other uses, including an app that helps you securely and easily
interact with the network, including creating Tezos transactions and baking
blocks.

This repository contains two Ledger applications:

  1. The "Tezos Baking" application is for baking Tezos: signing new blocks,
     endorsements, and denunciations. For more information about baking, see
     [*Benefits and Risks of Home Baking*](https://medium.com/@tezos_91823/benefits-and-risks-of-home-baking-a631c9ca745).
  2. The "Tezos Wallet" application is for making XTZ transactions, and
     everything you might want to use the Ledger Nano S for on Tezos besides baking.

It is possible to do all of these things without a hardware wallet, but using a
hardware wallet provides you better security against key theft.

We recommend you read this entire document to understand the commands available,
and which commands are most appropriate to your situation. This will require
judgment on how best to meet your needs, and this document will also provide
context to help you understand that.

The commands in these instructions have only been tested on Linux. If
you use any form of virtualization, e.g. docker or VirtualBox, please
consult the documentation of that virtualization system to determine
how to access USB from inside the virtualization, as that can be a
complicated and difficult process.

## Set up your Ledger device

Tezos recommends a hardware wallet called the Ledger Nano S. When you first get
it and set it up, part of the setup process is generating a keypair. The keypair
will be associated with a rather long seed phrase that you must write down and
keep securely. We'll discuss that seed phrase more below. You also set a PIN
code that allows you to unlock the device so that it will sign messages. You can
then install the Tezos app to use the Ledger Nano S to interact directly with the
Tezos network (see more about this in the app instructions, forthcoming).
However, your Ledger Nano S will ask for confirmation before it sends your keys to sign
transactions or blocks, and you must confirm by physically pushing a button on
the device, and that provides some security against an attacker taking control
over your keys.

### Protecting your key

The seed phrase is an encoding of your private key itself and can be used to
restore your key. If you lose your Ledger Nano S or destroy it somehow, you can buy a
new one and set it up with the seed phrase from your old one, hence restoring
your tokens.

Consequently, is is extremely important that you keep your seed phrase written
down somewhere safe. Losing it can mean you lose control of your account should
you, for example, lose your Ledger device. Keeping it somewhere a hacker could find it
(such as in a file on your internet-connected computer) means your private key
can fall into the wrong hands.

You will write it down on paper, along with your PIN, and store it. If you will
have a large amount of money, consider putting your paper in a safe or safe
deposit box, but at the very least keep it away from places where children,
dogs, housekeepers, obnoxious neighbors, could inadvertently destroy it.
Additionally, we recommend storing your PIN in a password manager.

Finally, if you will not be baking yourself, consider disconnecting your Ledger device
and closing your password manager when not in use.

### Protecting Your Key -- Further Advanced Reading

More advanced techniques for those interested in even more layers of security
or plausible deniability features should look at the
[Ledger's documentation on this](https://support.ledgerwallet.com/hc/en-us/articles/115005214529-Advanced-Passphrase-options).

Note that Ledger hardeware wallets with different seeds will appear to `tezos-client` to be
different Ledgers. Note also that it can change what key is authorized in
the Baking App. When using these features in a Ledger device used for baking,
please exit and re-start the Baking App right before baking is supposed to
happen, and manually verify that it displays the key you expect to bake for.

The Wallet App does not require such extra steps, and so these extra
protections are more appropriate for keys used for transaction than
they are for keys used for baking. If you do use these features, one
technique is that your tez be stored in the passphrase-protected and
deniable account, and that you delegate them to a baking account. This
way, the baking account won't actually store the vast majority of the tez.

### Ledger firmware update

To use these apps, you must be sure to have [up-to-date
firmware](https://support.ledgerwallet.com/hc/en-us/articles/360002731113)
on the Ledger Nano S. This code was tested with version
1.4.2. Please use the Ledger's tools, including their [Chrome
app](https://www.ledgerwallet.com/apps/manager), to do this. Note that
you can't talk to the Ledger OS itself while a Ledger app is running.

### udev rules (Linux only)

You need to set `udev` rules to set the permissions so that your user account
can access the Ledger device. This requires system administration privileges
on your Linux system.

#### Instructions for most distros (not including NixOS)

LedgerHQ provides a
[script](https://raw.githubusercontent.com/LedgerHQ/udev-rules/master/add_udev_rules.sh)
for this purpose. Download this script, read it, and run it as root:

```
$ wget https://raw.githubusercontent.com/LedgerHQ/udev-rules/master/add_udev_rules.sh
$ chmod +x add_udev_rules.sh
$ sudo ./add_udev_rules.sh
```

You might have to adjust which group this script uses, and modify the script for that
purpose. We recommend against piping `wget` into `sudo bash` without adjusting
the script to your computer's configuration.

Subsequently, unplug your Ledger device, and plug it in again for the changes to take
effect.

#### Instructions for NixOS

For NixOS, you can set the udev rules by adding the following to the NixOS
configuration file typically located at `/etc/nixos/configuration.nix`:

```nix
{
  services.udev.extraRules = ''
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2581", ATTRS{idProduct}=="1b7c", MODE="0660", GROUP="users"
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2581", ATTRS{idProduct}=="2b7c", MODE="0660", GROUP="users"
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2581", ATTRS{idProduct}=="3b7c", MODE="0660", GROUP="users"
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2581", ATTRS{idProduct}=="4b7c", MODE="0660", GROUP="users"
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2581", ATTRS{idProduct}=="1807", MODE="0660", GROUP="users"
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2581", ATTRS{idProduct}=="1808", MODE="0660", GROUP="users"
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2c97", ATTRS{idProduct}=="0000", MODE="0660", GROUP="users"
    SUBSYSTEMS=="usb", ATTRS{idVendor}=="2c97", ATTRS{idProduct}=="0001", MODE="0660", GROUP="users"
  '';
}
```

Depending on your system's settings, you may wish to replace `users` with
another group. Everyone in that group will get permissions for accessing the
Ledger device.

Once you have added this, run `sudo nixos-rebuild switch` to activate the
configuration, and unplug your Ledger Nano S and plug it in again for the changes to
take effect.

## Obtaining the Ledger apps

If you are using the [Nix package manager](https://nixos.org/nix/), you can skip
this section and the next one; go directly to
[Tezos baking platform](https://gitlab.com/obsidian.systems/tezos-baking-platform)
for simpler Nix-based installation. Then return to this document and continue
reading at *Using the Ledger apps*.

The easiest way to obtain the Tezos Ledger apps is to download the `.hex` files
from the [releases](https://github.com/obsidiansystems/ledger-app-tezos/releases)
page. After doing so, skip ahead to *Installing the apps onto your Ledger device*.
You will need to expand the releases tarball somewhere and copy the
baking.hex and wallet.hex files into the ledger-app-tezos directory.
If you want to compile the applications yourself, keep reading this section.

### Compiling the `.hex` files

The first thing you'll need to do is clone this repository:

```
$ git clone https://github.com/obsidiansystems/ledger-app-tezos.git
```

You will need to have the
[BOLOS SDK](http://ledger.readthedocs.io/en/latest/userspace/getting_started.html)
to use the Makefile, which can be cloned from Ledger's
[nanos-secure-sdk](https://github.com/LedgerHQ/nanos-secure-sdk) git repository.
You will also need to download two compilers for use with the SDK.
Note that these are specialized compilers to cross-compile for the ARM-based
platform of the Ledger Nano S; please don't use the versions of `clang` and `gcc` that
come with your system.

  * [CLANG](http://releases.llvm.org/4.0.0/clang+llvm-4.0.0-x86_64-linux-gnu-ubuntu-16.10.tar.xz)
  * [GCC](https://launchpadlibrarian.net/251687888/gcc-arm-none-eabi-5_3-2016q1-20160330-linux.tar.bz2)

All of the environment setup can be accomplished with the following commands.

Obtain the BOLOS SDK and the compilers it needs:

```
$ git clone https://github.com/LedgerHQ/nanos-secure-sdk
$ wget -O clang.tar.xz http://releases.llvm.org/4.0.0/clang+llvm-4.0.0-x86_64-linux-gnu-ubuntu-16.10.tar.xz
$ wget -O gcc.tar.bz2 https://launchpadlibrarian.net/251687888/gcc-arm-none-eabi-5_3-2016q1-20160330-linux.tar.bz2
```

Unzip the compilers and move them to appropriately-named directories:

```
$ mkdir bolos_env
$ tar -xJf clang.tar.xz --directory bolos_env
$ mv bolos_env/clang+llvm-4.0.0-x86_64-linux-gnu-ubuntu-16.10 bolos_env/clang-arm-fropi
$ tar -xjf gcc.tar.bz2 --directory bolos_env
```

Set environment variables:

```
$ export BOLOS_SDK=$PWD/nanos-secure-sdk
$ export BOLOS_ENV=$PWD/bolos_env
```

To build the Tezos Wallet app:

```
$ env BAKING_APP= make
$ mv bin/app.hex wallet.hex
```
If this results in an error message that includes this line (possibly repeatedly):

```
#include <bits/libc-header-start.h>
```
you may need to run:

```
$ sudo apt-get install libc6-dev gcc-multilib g++-multilib
```
and then re-run the `make` command.

Note that if you build *both* apps, you need to run `make clean` before building
the second one. So, to build both apps run:

```
$ env BAKING_APP= make
$ mv bin/app.hex wallet.hex
$ make clean
$ env BAKING_APP=Y make
$ mv bin/app.hex baking.hex
```

To build just the Tezos Baking App:

```
$ env BAKING_APP=Y make
$ mv bin/app.hex baking.hex
```

## Installing the apps onto your Ledger device

Ledger's primary interface for loading an app onto a Ledger device is a Chrome
app called [Ledger Manager](https://www.ledgerwallet.com/apps/manager). However,
it only supports a limited set of officially-supported apps like Bitcoin and
Ethereum, and Tezos is not yet among these. So you will need to use a
command-line tool called the
[BOLOS Python Loader](https://ledger.readthedocs.io/projects/blue-loader-python/en/0.1.16/index.html).

### Installing BOLOS Python Loader

Install `libusb` and `libudev`, with the relevant headers. On Debian-based
distros, including Ubuntu, the packages with the headers are suffixed with
`-dev`. Other distros will have their own conventions. So, for example, on
Ubuntu, you can do this with:

```
$ sudo apt-get install libusb-1.0.0-dev libudev-dev # Ubuntu example
```

Then, install `pip3`. You must install `pip3` for this and not `pip`. On Ubuntu:

```
$ sudo apt-get install python3-pip # Ubuntu example
```

Now, on any operating system, install `virtualenv` using `pip3`. It is important
to use `pip3` and not `pip` for this, as this module requires `python3` support.

```
$ sudo pip3 install virtualenv # Any OS
```

Then create a Python virtual environment (abbreviated *virtualenv*). You could
call it anything, but we shall call it "ledger". This will create a directory
called "ledger" containing the virtualenv:

```
$ virtualenv ledger # Any OS
```

Then, you must enter the `virtualenv`. If you do not successfully enter the `virtualenv`,
future commands will fail. You can tell you have entered the virtualenv when your prompt is
prefixed with `(ledger)`.

```
$ source ledger/bin/activate
```

Your terminal session -- and only that terminal session -- will now be in the
virtual env. To have a new terminal session enter the virtualenv, run the above
`source` command only in the same directory in the new terminal session.

### ledgerblue

Within the virtualenv environment -- making sure that `(ledger)` is showing up
before your prompt -- use pip to install the `ledgerblue`
[Python package](https://pypi.org/project/ledgerblue/).
This will install the Ledger Python packages into the virtualenv; they will be
available only in a shell where the virtualenv has been activated.

Note: We are installing this python package named "ledgerblue" even though we are working with Ledger Nano S hardware.
Do not skip this step even though you are not using Ledger Blue hardware.

```
$ pip install ledgerblue
```

If you have to use `sudo` or `pip3` here, that is an indication that you have
not correctly set up `virtualenv`. It will still work in such a situation, but
please research other material on troubleshooting `virtualenv` setup.

### Load the app onto the Ledger device

Next you'll use the installation script to install the app on your Ledger Nano
S.

The Ledger must be in the following state:

  * Plugged into your computer
  * Unlocked (enter your PIN)
  * On the home screen (do not have any app open)
  * Not asleep (you should not see *vires in numeris* is scrolling across the
    screen)

If you are already in an app or the Ledger Nano S is asleep, your installation process
will fail.

We recommend staying at your computer and keeping an eye on the Ledger device's screen
as you continue. You may want to read the rest of these instructions before you
begin installing, as you will need to confirm and verify a few things during the
process.

Still within the virtualenv, run the `./install.sh` command. This script is in
the root directory of this very repo, which means that in order to have it, you
must clone this repo and `cd` into the resulting directory:

```
$ git clone https://github.com/obsidiansystems/ledger-app-tezos.git
$ cd ledger-app-tezos/
```

This `./install.sh` script takes two parameters, the first of which is
the *name* of the application you are installing, and the second is the
path to the `app.hex` file. (Up to date `app.hex` files can be found in
the releases for this repo, and can be unpacked in the same directory that
contains `./install.sh`):

* If you are installing the baking app, we recommend using the name "Tezos
  Baking".

  ```
  $ ./install.sh "Tezos Baking" baking.hex
  ```

* If you are installing the transaction app, we recommend using the name "Tezos
  Wallet".

  ```
  $ ./install.sh "Tezos Wallet" wallet.hex
  ```

The first thing that should come up in your terminal is a message that looks
like this:

```
Generated random root public key : <long string of digits and letters>
```

Look at your Ledger Nano S screen and verify that the digits of that key match the
digits you can see on your Ledger device's screen. What you see on your Ledger device's screen
should be just the beginning and ending few characters of the longer string that
printed in your terminal.

You will need to push confirmation buttons on your Ledger Nano S a few times
during the installation process and re-enter your PIN code near the end of the
process. You should finally see the Tezos logo appear on the screen.

If you see the "Generated random root public key" message and then something
that looks like this:

```
Traceback (most recent call last):
File "/usr/lib/python3.6/runpy.py", line 193, in _run_module_as_main
<...more file names...>
OSError: open failed
```

the most likely cause is that your `udev` rules are not set up correctly, or you
did not unplug your Ledger between setting up the rules and attempting to
install. Please confirm the correctness of your `udev` rules.

To load a new version of the Tezos application onto the Ledger Nano S in the future,
you can run the command again, and it will automatically remove any
previously-loaded version.

### Removing Your App

If you'd like to remove your app, you can do this. In the virtualenv
described in the last sections, run this command:

```
$ python \
    -m ledgerblue.deleteApp \
    --targetId 0x31100003 \
    --appName "Tezos"
```

Replace the `appName` parameter "Tezos" with whatever app name you used when you
loaded the app onto the device.

Then follow the prompts on the Ledger Nano S screen.

### Confirming the Installation Worked

You should now have two apps, `Tezos Baking` and `Tezos Wallet`. The `Tezos
Baking` app should display also a `0` on the screen, which is the highest block
level baked so far (`0` in case of no blocks). The `Tezos Wallet` app will just
display `Tezos`.

## Registering the Ledger Nano S with the node

For the remainder of this document, we assume you have a Tezos node running and
`tezos-client` installed. Also, Docker has some issues working with the Ledger Nano S,
so unless you're willing to troubleshoot them, we don't recommend it.

Currently there are two other ways to do this:

  1. If you have the Nix package manager, use the
     [Tezos baking platform](https://gitlab.com/obsidian.systems/tezos-baking-platform).
  2. Build tezos from the tezos repo with [these instructions](http://doc.tzalpha.net/introduction/howto.html#get-the-sources).

Depending on how you build it, you might need to prefix `./` to your commands, and the names
of some of the binaries might be different.

### What is tezos-client

We can call the network at large "Tezos." Tezos consists of a bunch of nodes,
one of which is yours. Your node can be thought of as your gateway to the wider
network.

You can't do anything with the Ledger Nano S without using `tezos-client`. Tezos-client
is the program you use to access information about the network,
which you ultimately get through your node. See the
[command documentation](http://doc.tzalpha.net/api/cli-commands.html)
for the full array of features that tezos-client supports.

In summary:

* Tezos is the network
* We connect to the network through a node
* We access that node through tezos-client
* We store our client's keys on the ledger device

Note that `tezos-client` will not only not support certain commands unless the node is installed,
but the error messages for those commands will not even indicate that those commands are possible.
If a command documented here gives an `Unrecognized command` error, make sure you have a node
running.

### Side note about key generation

Every Ledger generates public and private keys for `ed25519`, `secp256k1`, or
`P-256` encryption systems based on a seed (represented by and encoded in
the words associated with that Ledger device) and a BIP32 ("hierarchical deterministic
wallet") path.

The same seed and BIP32 path will always result in the same key for the same
systems. This means that, to keep your Bitcoin app from knowing your Tezos keys,
and vice versa, different BIP32 paths have to be used for the same Ledger device. This
also means that, in order to sync two Ledger devices, you can set them to the same
seed, represented as 24 or some other number of natural language words (English
by default).

All Tezos BIP32 paths begin with `44'/1729'` (the `'` indicates it is
"hardened").  Which Ledger device is intended to be used, as well as choice of
encryption system, is indicated by a root key hash, the Tezos-specific base58
encoding of the hash of the public key at `44'/1729'` on that Ledger device. Because
all Tezos paths start with this, in `tezos-client` commands it is implied.

### Importing the key from the Ledger Nano S

This section must be done regardless of whether you're going to be baking or
only using the Tezos Wallet application.

Please run, with a Tezos app (either Tezos Baking or Tezos Wallet will do) open
on your Ledger Nano S:

```
$ tezos-client list connected ledgers
```

The output of this command includes six Tezos addresses derived from the secret
stored on the device, via different signing curves and BIP32 paths.

```
Found a valid Tezos application running on Ledger Nano S at [0001:0014:00].

To add the root key of this ledger, use one of
  tezos-client import secret key ledger_<...>_ed ledger://tz1Rctvczeh7WK91NfT9LUSrYJFceyhJKUQN # Ed25519 signature
  tezos-client import secret key ledger_<...>_secp ledger://tz2EhYFRUhY4YiinHYxb7PncETZXP9BfRSPj # Secp256k1 signature
  tezos-client import secret key ledger_<...>_p2 ledger://tz3RZ5JvhuSAg3i4qBSaoCKqXwmqhswqeNgF # P-256 signature
Each of these tz* is a valid Tezos address.

To use a derived address, add a hardened BIP32 path suffix at the end of the URI.
For instance, to use keys at BIP32 path m/44'/1729'/0'/0', use one of
  tezos-client import secret key ledger_<...>_ed_0_0 "ledger://tz1Rctvczeh7WK91NfT9LUSrYJFceyhJKUQN/0'/0'"
  tezos-client import secret key ledger_<...>_secp_0_0 "ledger://tz2EhYFRUhY4YiinHYxb7PncETZXP9BfRSPj/0'/0'"
  tezos-client import secret key ledger_<...>_p2_0_0 "ledger://tz3RZ5JvhuSAg3i4qBSaoCKqXwmqhswqeNgF/0'/0'"
In this case, your Tezos address will be a derived tz*.
It will be displayed when you do the import, or using command `show ledger path`.
```

The first three are addressed constructed directly from the root key for each
encryption system (`ed25519`, `secp256k1`, or P-256). The second three show you
examples of how to append a BIP32 path to the root key to construct a new
address.

The Ledger Nano S does not currently support non-hardened path components. All
components of all paths must be hardened, which is indicated by following them
with a `'` character. This character may need to be escaped from the shell
through backslashes `\` or double-quotes `"`.

You'll need to choose one of the six commands starting with `tezos-client import
secret key ...` to run.

  * `ed25519` is the standard recommended curve
  * If you know you're always going to use only one key, use a root key; you can
    only have one root account per ledger per signing curve. If you might want
    to leave open the possibility of using your Ledger for multiple keys,
    use a BIP32 path. We recommend the latter.

If you use a BIP32 path, be sure to write it down. You need the full address to
use your tez. This means that if you lose all your devices and need to set
everything up again, you will need three things:

  1. The mnemonic phrase
  2. Which signing curve you chose
  3. The BIP32 path, if you used one

The `tezos-client import secret key` operation copies only the public key; it
says "import secret key" to indicate that the Ledger's secret key will be
considered available for signing from them on, but it does not leave the Ledger.

This sends a BIP32 path to the Ledger. You then need to click a button on the
Ledger device, and the Ledger then sends the public key back to the computer.

After you perform this step, if you run the `list known addresses` command, you
should see the key you chose in the list:

```
$ tezos-client list known addresses
ledger_<...>_ed_0_0: tz1ccbGmKKwucwfCr846deZxGeDhiaTykGgK (ledger sk known)
```

We recommend reading as much as possible about BIP32 to ensure you fully understand
this.

## Using the Tezos Wallet application

This application and the Tezos Baking Application constitute complementary apps
for different use cases -- which could be on paired Ledger hardware wallets and therefore use
the same key, or which could also be used in different scenarios for different
accounts. Baking is rejected by this app.

The "provide address" command on the Tezos Wallet application shows the address
the first time the command is run for any given session. Subsequently, it
provides the address without prompting. To display addresses again, exit the
Wallet Application and restart it. This is again provided for testing/initial
set up purposes.

The sign command for the Wallet Application prompts every time for transactions
and other "unsafe" operations, with the generic prompt saying "Sign?" We hope to
eventually display more transaction details along with this. When block headers
and endorsements are sent to the Ledger, they are rejected silently as if the
user rejected them.

### Faucet (zeronet only)

On zeronet, you will need to use the [Tezos Faucet](https://faucet.tzalpha.net/)
to obtain some tez. Tell them you're not a robot, then click "Get alphanet tz."
It works on zeronet (even though the URL says alphanet).

Run the following command, where `<your-name>` is some alias you want to use for
this wallet, and `tz1<...>.json` is the name of the file you just downloaded
from the faucet.

```
$ tezos-client activate account <your-name> with ~/downloads/tz1<...>.json
Node is bootstrapped, ready for injecting operations.
Operation successfully injected in the node.
Operation hash is 'onxJStKxK1oMPgGskkzc2gDBDyKeQ7CbBYTrcK4TMMySvKZq6vF'.
Waiting for the operation to be included...
Operation found in block: BMRjW94ge499sCPAMUTvrp3ku2UjWy9kB2LsjtuJhL1bkcQ85Ny (pass: 2, offset: 0)
This sequence of operations was run:
  Genesis account activation:
    Account: tz1Vntj2aVpqcQEeHq2CEmNrSGw8finvbFcX
    Balance updates:
      tz1Vntj2aVpqcQEeHq2CEmNrSGw8finvbFcX ... +ꜩ66835.212314

Account <your-name> (tz1Vntj2aVpqcQEeHq2CEmNrSGw8finvbFcX) activated with ꜩ66835.212314.
```

You can then check your account balance like this:

```
$ tezos-client get balance for <your-name>
66835.212314 ꜩ
```

### Transfer

Now transfer the balance to the account whose key resides on your Ledger:

```
$ tezos-client transfer 66000 from chris-martin2 to ledger_<...>_ed_0_0
```

### Further transaction details

In general, to send tez, you'll need to:

  * Have a node running
  * Open the Tezos Wallet app on your Ledger
  * Know the alias of your account or its public key hash
  * Know the public key hash of the account you are sending tez to

The command you run has the form:

```
tezos-client transfer QTY from SRC to DST
```

  * `QTY` is the amount of tez. It's best to not include commas and to include 6
    decimal points (ie. 1000000.000000). If you'd prefer to include commas, you can:
    `1,000,000.000,000`.
  * `SRC` is the source, or where the money is coming from. This should be your
    alias or public key has.
  * `DST` is the destination, or where the money is going. You should use the
    public key hash, as your computer likely doesn't know any aliases for that
    account.

Some options which you can consider:

  * `--fee <amount>` - The fee defaults to 0.05 tez. If you'd like to select
    another amount, either because you think that's too high or the network is
    crowded and a higher fee is needed to ensure it goes through, you can
    include this with the amount of fee you want to pay (ie. `--fee 0.05` for
    the default).
  * `-D` or `--dry-run` - Use this if you just want to display what would happen
    and not actually do the transaction.
  * `-G` or `--gas-limit` - This sets the gas limit of the transaction instead.

There are other options which you can read up about more in the docs, but these
are the main ones you'd potentially want to use when just sending tez to
someone.

### Delegation

If you want to delegate tez controlled by a Ledger Nano S to another account to
bake, that requires the Wallet App. This is distinct from registering the Ledger
account itself to bake, which is also called "delegation," and which is covered
in the section on the baking app below.

To delegate tez controlled by a Ledger Nano S to someone else,
you must first originate an account. Please read more
about this in the Tezos document, [How to bake on the
alphanet](http://doc.tzalpha.net/introduction/alphanet.html#how-to-bake-on-the-alphanet), to
understand why this is necessary and the semantics of delegation.

To originate an account, the command is:
```
$ tezos-client originate account <NEW> for <MGR> transferring <QTY> from <SRC> --delegatable
```

  * `NEW` is the alias you will now give to the originated account. Only originted accounts can
     be delegated, and even then only if originated with this `--delegatable` flag.
  * `MGR` is the name of the key you will use to manage the account. If you want to manage it
    with a Ledger, it should be an existing imported key from the Ledger.
  * `QTY` is the initial amount of tez to give to the originated account.
  * `SRC` is the account where you'll be getting the tez from.

Subsequently, every transaction made with `<NEW>` will require the Ledger Nano S mentioned in `<MGR>`
to sign it. This is done with the wallet application, and includes setting a delegate with:

```
$ tezos-client set delegate for <NEW> to <DELEGATE>
```

Originated accounts have names beginning with `KT1` rather than `tz1`, `tz2` or `tz3`.

## Using the Tezos Baking Application

The Tezos Baking Application supports 3 operations:

  1. Authorize/get public key
  2. Reset high water mark
  3. Sign

It will only sign block headers and endorsements, as the purpose of the baking
app is that it cannot be co-opted to perform other types of operations (like
transferring XTZ). If a Ledger device is running with the Tezos Baking Application, it
is the expectation of its owner that no transactions will need to be signed with
it. To sign transactions with that Ledger device, you will need to switch it to using
the Tezos Wallet application, or have the Tezos Wallet application installed on
a paired Ledger. Therefore, if you have a larger stake and bake frequently, we
recommend the paired Ledger approach. If, however, you bake infrequently and can
afford to have your baker offline temporarily, then switching to the Tezos
Wallet application on the same Ledger device should suffice.


### Start the baking daemon

```
$ tezos-baker-alpha run with local node ~/.tezos-node ledger_<...>_ed_0_0
```

Alternatively, it might be called `tezos-alpha-baker`.

This won't actually be able bake successfully yet until you run the rest of
these setup steps. This will run indefinitely, so you might want to do it in
a dedicated terminal or in a `tmux` or `screen` session.

You will also want to start the endorser and accuser daemons:

```
$ tezos-endorser-alpha run ledger_<...>_ed_0_0
$ tezos-accuser-alpha run
```

Alternatively, they might might be called `tezos-alpha-endorser` and `tezos-alpha-accuser`.
Again, each of these will run indefinitely, and each should be in its own terminal
`tmux`, or `screen` window.

### Authorize public key

You need to run a specific command to authorize a key for baking. Once a key is
authorized for baking, the user will not have to approve this command again. If
a key is not authorized for baking, signing endorsements and block headers with
that key will be rejected. This authorization data is persisted across runs of
the application.  Only one key can be authorized for baking per Ledger device at a
time.

In order to authorize a public key for baking, you can do either of the
following:

  * Use the APDU for authorizing a public key.

    The command for this is as follows:

    ```
    $ tezos-client authorize ledger to bake for <SIGNATURE>
    ```

    This only authorizes the key for baking on the Ledger Nano S, but does
    not inform the blockchain of your intention to bake. This might
    be necessary if you re-install the app, or if you have a different
    paired Ledger that you are using to bake for the first time.

  * Have the baking app sign a self-delegation. This is explained in the next section.

### Registering as a Delegate

In order to bake from the Ledger account you need to register the key as a
delegate. This is formally done by delegating the account to itself. As a
non-originated account, an account directly stored on the Ledger device can only
delegate to itself.

Open the Tezos Baking Application on the Ledger Nano S, and then run this:

```
$ tezos-client register key ledger_<...>_ed_0_0 as delegate
```

This command is intended to inform the blockchain itself of your intention to
bake with this key. It can be signed with either the Wallet App or the Baking
App, but if you sign it with the Baking App, it also implies to the Ledger
that you want to authorize that key for baking on the Ledger as well.

Authorizing a key for baking on a specific Ledger device and authorizing it for
baking in general on the blockchain are two distinct authorizations. This
command will do both of them if signed with the appropriate baking app,
whereas the `authorize ledger` command in the previous section will
only do it for the in question, which is appropriate if it is already
authorized to bake on the blockchain.

The `register key` command is equivalent to:

```
$ tezos-client set delegate for ledger_<...>_ed_0_0 to ledger_<...>_ed_0_0
```

The Baking App only signs self-delegations; the Wallet App is needed to sign
delegations of originated accounts controlled by a Ledger hardware wallet. The Baking App
also only signs delegations with fees less than 0.05 ꜩ; to sign those with
more, you must use the Wallet App to authorize baking for the blockchain,
and the command in the previous section to authorize baking for the Ledger Nano S,
which is always available as an alternative to signing this with the Baking App.

### Sign

The sign operation is for signing block headers and endorsements.

Block headers must have monotonically strictly increasing levels; that is, each
block must have a higher level than all previous blocks signed with the Ledger device.
This is intended to prevent double baking at the Ledger level, as a security
measure against potential vulnerabilities where the computer might be tricked
into double baking. This feature will hopefully be a redundant precaution, but
it's implemented at the Ledger level because the point of the Ledger Nano S is to not
trust the computer. The current High Water Mark (HWM) -- the highest level to
have been baked so far -- is displayed on the Ledger device screen, and is also
persisted between runs of the device.

The sign operation will be sent to the Ledger device by the baking daemon when
configured to bake with a Ledger key. The Ledger uses the first byte of the
information to be signed -- the magic number -- to tell whether it is a block
header (which is verified with the High Water Mark), an endorsement (which is
not), or some other operation (which it will reject, unless it is a
self-delegation).

With the exception of self-delegations, as long as the key is configured and the
high water mark constraint is followed, there is no user prompting required for
signing. The baking app will only ever sign without prompting or reject an
attempt at signing; this operation is designed to be used unsupervised.

As mentioned, the only exception to this is self-delegation. This will prompt
and display the public key hash of the account. This will authorize the key for
baking on the Ledger. This is independent of what the signed self-delegation
will then do on the blockchain. If you are baking on a new Ledger Nano S, or have
reinstalled the app, you might have to sign a self-delegation to authorize the
key on the Ledger Nano S, even if you are already registered for baking on the
block-chain. This will also have to be done if you have previously signed this
with the wallet app.

### Reset

There is some possibility that the HWM will need to be reset. This could be due
to the computer somehow being tricked into sending a block with a high level to
the Ledger (either due to an attack or a bug on the computer side), or because
the owner of the Ledger wants to move from the real Tezos network to a test
network, or to a different test network. This can be accomplished with the reset
command.

This is intended as a testing/debug tool and as a possible recovery from
attempted attacks or bugs, not as a day to day command. It requires an explicit
confirmation from the user, and there is no specific utility to send this
command. To send it manually, you can use the following command line:

`tezos-client set ledger high water mark for "ledger://<tz...>/" to <HWM>`

`<HWM>` indicates the new high water mark to reset to. If you are joining a new
test network, `0` is a fitting number, as then all blocks will be allowed again.
You can also set it to the most recently baked level on that test net.

If you are not physically present at your computer, but are curious what the
Ledger's high water mark is, you can run:

`tezos-client get ledger high water mark for "ledger://<tz...>/`

## Upgrading

When you want to upgrade to a new version, whether you built it yourself from source
or whether it's a new release of the `app.hex` files, use the same commands as you did
to originally install it. As the keys are generated from the Ledger device's seed and the
derivation paths, you will have the same keys with every version of this Ledger app,
so there is no need to re-import the keys with `tezos-client`.

### Special Upgrading Considerations for Bakers

If you've already been baking on an old version of the Baking App, the new version will
not remember which key you are baking with nor the High Water Mark. You will have to re-run
this command to remind the Ledger what key you intend to authorize for baking:

```
$ tezos-client authorize ledger to bake for <SIGNATURE>
```

You can also set the High Water Mark to the level of the most recently baked block with:

```
tezos-client set ledger high water mark for "ledger://<tz...>/" to <HWM>
```

This will require the correct URL for the Ledger Nano S acquired from:

```
tezos-client list connected ledgers
```

## Troubleshooting

### unexpected seq num

```
$ client/bin/tezos-client list connected ledgers
Fatal error:                                                                                                                                        Header.check: unexpected seq num
```

This means you do not have the Tezos app open on your Ledger Nano S.

### No device found

```
$ tezos-client list connected ledgers
No device found.
Make sure a Ledger Nano S is connected and in the Tezos Wallet app.
```

In addition to the possibilities listed in the error message, this could also
mean that your udev rules are not set up correctly.

### Unrecognized command

If you see an `Unrecognized command` error, it might be because there is no node for `tezos-client`
to connect to. Please ensure that you are running a node. `ps aux | grep tezos-node` should display
the process information for the current node. If it displays nothing, or just displays a `grep`
command, then there is no node running on your machine.

### Ledger App Crashes

If the Ledger app crashes when you load it, there are two primary causes:

  * Quitting the `tezos-client` process before the Ledger responds. Even if you meant to cancel
    the operation in question, cancel it from the Ledger before pressing Ctrl-C, otherwise you
    might have to restart the Ledger.
  * Out of date firmware: If the Ledger app doesn't work at all, make sure you are running firmware
    version 1.4.2.
