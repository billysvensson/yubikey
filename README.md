# Setup Yubikey for SSH, Git and Email on Linux
## Introduction
This tutorial will help you setup a Yubikey to be used for SSH authentication, git signing and email signing/encryption on Linux. We will also export the keypair for offline storage and show how you restore the keys on to a new Yubikey if your main Yubikey gets destroyed.

## Installation
1. Install the following packages:
..* gnupg2
..* pcsc-tools
..* ccid
..* libusb-compat
..* vim
..* yubitouch.sh (https://gist.github.com/a-dma/797e4fa2ac4b5c9024cc)

## Configure system
2. Add `export SSH_AUTH_SOCK=/run/user/$UID/gnupg/S.gpg-agent.ssh` to your `.bashrc` or `.zshrc`.
3. Edit `.gnupg/gpg-agent.conf` and add:
```
pinentry-program /usr/bin/pinentry
enable-ssh-support
default-cache-ttl 3600
max-cache-ttl 3600
```
4. Edit `.gnupg/gpg.conf` and add:
```
use-agent
personal-cipher-preferences AES256
personal-digest-preferences SHA512
cert-digest-algo SHA512
default-preference-list SHA512 AES256 ZLIB BZIP2 ZIP Uncompressed
```
5. Add `gpg-agent --daemon` to your startup script.

## Generate keypair
6. Run `gpg --expert --full-generate-key`, select option `8 - RSA (set your own capabilities)` and generate a new RSA 4096 bits key with only Certify capability enabled, your name and email.
7. Run `gpg2 --expert --edit-key your@email.here`.
8. Type `addkey` and chose `8 - Select own` and toggle capabilities so that only [E] encrypt is enabled.
9. Repeat previous step two times and add one key with [S] Sign enabled and another key with [A] Authenticate enabled.
10. Type `save`.

## Set Yubikey passwords
11. Insert your Yubikey and run `gpg2 --card-edit`.
12. Type `admin`, then `passwd` and update both the password and the admin password.
13. Type `quit` when youre done.

## Export keys to flashdrive
14. Insert and mount your flashdrive where you want to keep the private key backups.
15. Export public key with 'gpg2 --armor --export your@email.here > yourname.public.gpg-key'.
16. Export private key with `gpg2 --armor --export-secret-keys your@email.here > yourname.private.gpg-key`.
17. Export private subkeys with `gpg2 --armor --export-secret-subkeys your@email.here > yourname.subkeys.private.gpg-key`.
18. Export public ssh key with `gpg2 --export-ssh-key your@email.here > yourname.public.ssh-key`.

## Move keys to Yubikey
19. Run `gpg2 --expert --edit-key your@email.here`.
20. Type `key 2` followed by `keytocard`.
21. Move encrypt, sign and authenticate keys to the Yubikey and type `save` when youre done.
22. Verify that the keys have been moved with the command `gpg2 --card-status`.
23. Delete the local keys with `gpg2 --delete-secret-and-public-key your@email.here`.

## Enable tuch-to-sign for all Yubikey actions
24. ./yubitouch.sh sig on
25. ./yubitouch.sh aut on
26. ./yubitouch.sh dec on

## Restore keys to new Yubikey (in case of key failiure)
1. Remove the old key from `~/.gnupg/private-keys-v1.d/`.
2. Import keys with `gpg2 --import yourname.private.gpg-key yourname.public.gpg-key`.
3. Follow the steps from the section "Move keys to Yubikey".
