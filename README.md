OnRISC ELBE based Debian BSP
============================

Debian based E.mbedded L.inux B.uild E.nvironment (ELBE) uses an XML based
configuration file to create a root file system. This configuration file covers
various root file system related aspects like package and repository
management, binary image creation (both SD card and MTD based devices are
supported) and file system finetuning (create, delete or rename files, foldera
etc.).

For further information visit project's hope page: https://elbe-rfs.org/

ELBE Installation
-----------------

You'll need a Debian Jessie host in order to use ELBE. Perform following steps:

1. `apt install python python-mako python-lxml python-apt python-gpgme python-pyme python-suds tmux qemu-utils qemu-kvm p7zip-full make reprepro`
2. `git clone https://github.com/visionsystemsgmbh/vscom-elbe.git`
3. `git clone https://github.com/Linutronix/elbe.git`
4. `cd elbe`
5. `git checkout devel/elbe-2.0`

Before you can use ELBE as a regular user you need to add this user to the
kvm/libvrt groups:

    adduser <youruser> kvm
    adduser <youruser> libvirt

Setup own Debian Package Repository
-----------------------------------

First of all you'll need to create a public key for repository signing. Invoke
following command and use default settings.

    gpg --gen-key

In our example we assume that the user is "User", e-mail is "user@example.com"
and comment is "Nothing".

Create a folder `debs` near the `elbe` and `vscom-elbe` folders. Change to it
and perform:

1. `gpg -a -o user@example.com.gpg.key --export user@example.com`
2. `mkdir conf`
3. create a file `conf/distributions` with following content


    Origin: apt.ownelbe.com
    Label: apt repository
    Codename: jessie
    Architectures: armhf
    Components: main
    Description: Own ELBE Debian repo
    SignWith: yes

Create folder `debs-bin` near `elbe` and download latest `kernel`, `libonrisc`
and `libsoc` packages from
ftp://ftp.visionsystems.de/pub/multiio/OnRISC/Baltos/deb/.

Now you're ready to create a Debian repository structure using `reprepro`:

1. `cd debs`
2. `reprepro --ask-passphrase -Vb . includedeb jessie ../debs-bin/*.deb`

`reprepro` will ask you the same password you gave during the public key
creation. To make this repository available over network perform:

    python -m SimpleHTTPServer 8888

This will start a HTTP server listening on port 8888.

Building Minimal Debian Image
-----------------------------

`vscom-elbe` holds various image configurations under `configs`. You'll need to
edit `configs/armhf-vscom-baltos-minimal.xml` and provide the IP address of
the host running the package repository you've already created. Just replace
`localhost` with the proper IP address.

    <url-list>
            <url>
                    <binary>http://localhost:8888 jessie main</binary>
                    <key>http://localhost:8888/user@example.com.gpg.key</key>
            </url>
    </url-list>

To create minimal Debian image perform:

1. `cd ../elbe`
2. `./elbe initvm --skip-build-bin --skip-build-sources create --directory=initvm`
3. `./elbe initvm --skip-build-bin --skip-build-sources submit --directory=initvm ../vscom-elbe/configs/armhf-vscom-baltos-minimal.xml`

Your SD card image together with build logs can be found under
`elbe-build-timestamp`. Extract `sdcard.img` from `sdcard.img.gz` and burn
it to your card.

Please refer to https://elbe-rfs.org/docs/sphinx/article-elbeoverview-en.html
for more details about XML configuration files and the whole building process.

Related Links
-------------

1. `reprepro` man page: https://mirrorer.alioth.debian.org/reprepro.1.html
