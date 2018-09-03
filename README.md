RouterOS Scripts
================

[RouterOS](https://mikrotik.com/software) is the operating system developed
by [MikroTik](https://mikrotik.com/aboutus) for networking tasks. This
repository holds a number of [scripts](https://wiki.mikrotik.com/wiki/Manual:Scripting)
to manage RouterOS devices or extend their functionality.

*Use at your own risk!*

Initial setup
-------------

The update script does server certificate verification, so first step is to
download the certificates.

    [admin@MikroTik] > / tool fetch "https://letsencrypt.org/certs/isrgrootx1.pem.txt"
          status: finished
      downloaded: 1KiBC-z pause]
           total: 1KiB
        duration: 1s

    [admin@MikroTik] > / tool fetch "https://letsencrypt.org/certs/letsencryptauthorityx3.pem.txt"
          status: finished
      downloaded: 1KiBC-z pause]
           total: 1KiB
        duration: 1s

Note that the commands above do *not* verify server certificate, so if you
want to be safe download with your workstations's browser and transfer the
files to your MikroTik device.

* [ISRG Root X1](https://letsencrypt.org/certs/isrgrootx1.pem.txt)
* [Let's Encrypt Authority X3](https://letsencrypt.org/certs/letsencryptauthorityx3.pem.txt)

Then we import the certificates.

    [admin@MikroTik] > /certificate import file-name=isrgrootx1.pem.txt passphrase=""
         certificates-imported: 1
         private-keys-imported: 0
                files-imported: 1
           decryption-failures: 0
      keys-with-no-certificate: 0

    [admin@MikroTik] > /certificate import file-name=letsencryptauthorityx3.pem.txt passphrase=""
         certificates-imported: 1
         private-keys-imported: 0
                files-imported: 1
           decryption-failures: 0
      keys-with-no-certificate: 0

Now let's download the main scripts, add them in configuration and remove the files.

    [admin@MikroTik] > / tool fetch check-certificate=yes-without-crl dst-path="script-updates/global-config" "https://git.eworm.de/cgit.cgi/routeros-scripts/plain/global-config\?h=pre-6-43"
          status: finished
      downloaded: 1KiBC-z pause]
           total: 1KiB
        duration: 0s

    [admin@MikroTik] > / tool fetch check-certificate=yes-without-crl dst-path="script-updates/script-updates" "https://git.eworm.de/cgit.cgi/routeros-scripts/plain/script-updates\?h=pre-6-43"
          status: finished
      downloaded: 1KiBC-z pause]
           total: 1KiB
        duration: 1s

    [admin@MikroTik] > / system script add name=global-config source=[ / file get script-updates/global-config contents ]
    [admin@MikroTik] > / file remove script-updates/global-config
    [admin@MikroTik] > / system script add name=script-updates source=[ / file get script-updates/script-updates contents ]
    [admin@MikroTik] > / file remove script-updates/script-updates

The configuration needs to be tweaked for your needs. Make sure not to send your mails to `mail@example.com`!

    [admin@MikroTik] > / system script edit global-config source

And finally load the configuration and add a scheduler.

    [admin@MikroTik] > / system script run global-config
    [admin@MikroTik] > / system scheduler add name=global-config start-time=startup on-event=global-config

Updating scripts
----------------

To update existing script just run `script-updates`.

    [admin@MikroTik] > / system script run script-updates

Adding a script
---------------

To add a script from the repository create a configuration item first, then
update scripts to fetch the source.

    [admin@MikroTik] > / system script add name=check-routeros-update
    [admin@MikroTik] > / system script run script-updates

Scheduler and events
--------------------

Most scripts are designed to run regularly from
[scheduler](https://wiki.mikrotik.com/wiki/Manual:System/Scheduler). We just
added `check-routeros-update`, so let's run it every hour to make sure not to
miss an update.

    [admin@MikroTik] > / system scheduler add name=check-routeros-update interval=1h on-event=check-routeros-update

Some events can run a script. If you want your DHCP hostnames to be available
in DNS use `dhcp-to-dns` with the events from dhcp server. For a regular
cleanup add a scheduler entry.

    [admin@MikroTik] > / system script add name=dhcp-to-dns
    [admin@MikroTik] > / system script run script-updates
    [admin@MikroTik] > / ip dhcp-server set lease-script=dhcp-to-dns [ find ]
    [admin@MikroTik] > / system scheduler add name=dhcp-to-dns interval=5m on-event=dhcp-to-dns

There's much more to explore... Have fun!

### Upstream

URL:
[GitHub.com](https://github.com/eworm-de/routeros-scripts#routeros-scripts)

Mirror:
[eworm.de](https://git.eworm.de/cgit.cgi/routeros-scripts/about/)
[GitLab.com](https://gitlab.com/eworm-de/routeros-scripts#routeros-scripts)

