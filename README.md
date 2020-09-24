OnRISC ELBE based Debian BSP
============================

Debian based E.mbedded L.inux B.uild E.nvironment (ELBE) uses an XML based
configuration file to create a root file system. This configuration file covers
various root file system related aspects like package and repository
management, binary image creation (both SD card and MTD based devices are
supported) and file system finetuning (create, delete or rename files, folders
etc.).

For further information visit project's home page: https://elbe-rfs.org/

ELBE Installation
-----------------

You'll need a Debian Buster host in order to use ELBE. Perform the following steps:

1. `mkdir /home/user/debian`
2. `cd /home/user/debian`
3. `apt install python python-mako python-lxml python-apt python-gpgme python-pyme python-suds tmux qemu-utils qemu-kvm p7zip-full libvirt-bin make`
4. `git clone https://github.com/visionsystemsgmbh/vscom-elbe.git`
5. `git clone https://github.com/Linutronix/elbe.git`
6. `cd elbe`
7. `git checkout v12.3`

Before you can use ELBE as a regular user you need to add this user to the
kvm/libvrt groups:

    adduser <youruser> kvm
    adduser <youruser> libvirt

Set up own Debian Package Repository
-----------------------------------

First of all, you'll need to create a public key for repository signing. Invoke
the following command and use default settings.

    gpg --gen-key

In our example, we assume that the e-mail is configured as "user@example.com".

Download and install `freight` from https://github.com/freight-team/freight.
Right now, `freight` doesn't support packages in quilt format. Perform the
following actions to install the custom version:

1. `git clone https://github.com/yegorich/freight.git`
2. `cd freight`
3. `git checkout support-quilt-format`
4. `make install`

Create the following folders:

* `/home/user/debian/freight/lib`
* `/home/user/debian/freight/cache`

Create a file `~/.freight.conf` with the following content:

    GPG="user@example.com"
    VARLIB=/home/user/debian/freight/lib
    VARCACHE=/home/user/debian/freight/cache
    ARCHS=armhf

Create folder `debs-bin` near `elbe` and download at least `kernel` and
`libonrisc` packages from
ftp://ftp.visionsystems.de/pub/multiio/OnRISC/Baltos/deb/buster.

Now you're ready to create a Debian repository structure using `freight`:

1. `cd /home/user/debian/debs-bin`
2. `freight add *.deb apt/buster`
3. `freight cache`

`freight` will ask you the same password you gave during the public key
creation. To make this repository available over network perform:

1. `cd /home/user/debian/freight/cache`
2. `python -m SimpleHTTPServer 8888`

This will start a HTTP server listening on port 8888.

Building Minimal Debian Image
-----------------------------

`vscom-elbe` holds various image configurations under `configs`. You'll need to
edit `configs/armhf-vscom-baltos-minimal.xml` and provide the IP address of
the host running the package repository you've already created. Just replace
`localhost` with the proper IP address.

    <url-list>
            <url>
                    <binary>http://localhost:8888 buster main</binary>
                    <key>http://localhost:8888/user@pubkey.gpg</key>
            </url>
    </url-list>

To create a minimal Debian image, perform:

1. `cd /home/user/debian/elbe`
2. `./elbe initvm --devel create --directory=initvm`
3. `./elbe initvm --skip-build-bin submit --directory=initvm ../vscom-elbe/configs/armhf-vscom-baltos-minimal.xml`

Your SD card image together with build logs can be found under
`elbe-build-timestamp`. Extract `sdcard.img` from `sdcard.img.gz` and burn
it to your card (you do not need to extract the image if you use
https://www.balena.io/etcher tool).

System overlay files are stored in the `armhf-vscom-baltos-minimal.xml` file as
a `tar.bz2` archive. https://elbe-rfs.org/docs/sphinx/article-quickstart.html#advanced-usage
describes how to access and modify these files.

Please refer to https://elbe-rfs.org/docs/sphinx/article-elbeoverview-en.html
for more details about XML configuration files and the whole building process.

Related Links
-------------

1. `freight` man page: http://freight-team.github.io/freight/freight.5.html
