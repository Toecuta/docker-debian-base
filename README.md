# Debian Working System for Docker

This is a simple set of images that transform the standard Docker
Debian environment into one that provides more traditional full
Unix APIs (including syslog, zombie process collection, etc.)

Despite this, they are all very small, both in terms of disk and RAM usage.

You can find a [description of the motivation for these images](http://changelog.complete.org/archives/9794-fixing-the-problems-with-docker-images) on my blog.

This is loosely based on the concepts, but not the code, in the
[phusion baseimage-docker](https://github.com/phusion/baseimage-docker).
You can look at that link for additional discussion on the motivations.

You can find the source and documentation at the [Github page](https://github.com/jgoerzen/docker-debian-base)
and automatic builds are available from [my Docker hub page](https://hub.docker.com/jgoerzen/).

This image uses sysvinit instead of systemd, not because of any
particular opinion on the merits of them, but rather because
sysvinit does not require any kind of privileged Docker
or cgroups access.  

Here are the images I provide from this repository:

- jgoerzen/debian-base-minimal
  - Provides working sysvinit, syslogd, cron, anacron, at, and logrotate.
  - syslogd is configured to output to the docker log system by default.
- jgoerzen/debian-base-standard - everything above, plus:
  - Utilities: less, nano, vim-tiny, man-db (for viewing manpages), net-tools
  - Email: exim4-daemon-light, mailx
  - Network: netcat-openbsd, socat, openssl, ssh, telnet (client)
- jgoerzen/debian-base-security - everything above, plus:
  - automated security patches using unattended-upgrades and needrestart
  - debian-security-support
- jgoerzen/debian-base-apache - everything above, plus:
  - apache2 plus utilities: ssl-cert
  - LetsEncrypt options: certbot, acme-tiny
- jgoerzen/debian-base-apache-php - everything above, plus:
  - libapache2-mod-php (mod-php5 on jessie)

Memory usage at boot (stretch):

- jgoerzen/debian-bas-eminimal: 6MB
- jgoerzen/debian-base-standard: 11MB
- jgoerzen/debian-base-security: 11MB

These images are autobuilt for jessie, stretch, and sid.

# Install

You can install with:

    docker pull jgoerzen/debian-base-whatever

Your Dockerfile should use CMD to run `/usr/local/bin/boot-debian-base`.

When running, use `-t` to enable the logging to `docker logs`

# Environment Variables

This environment variable is available for your use:

 - `DEBBASE_SYSLOG` defaults to `stdout`, which redirects all syslog activity
   to the Docker infrastructure.  If you instead set it to `internal`, it will
   use the default Debian configuration of logging to `/var/log` within the
   container.  The configuration is applied at container start time by
   adjusting the `/etc/syslog.conf` symlink to point to either `syslog.conf.internal` or
   `syslog.conf.stdout`.  `syslog.conf.internal` is the default from the system.
   `dpkg-divert` is used to force all packages' attempts to write to `/etc/syslog.conf`
   to instead write to `/etc/syslog.conf.internal`.
- `DEBBASE_TIMEZONE`, if set, will configure the `/etc/timezone` and `/etc/localtime`
  files in the container to the appropriate timezone.

# Container initialization

Executables or scripts may be placed in `/usr/local/preinit`, which will be executed
at container start time by `run-parts` prior to starting init.  These can
therefore perform container startup steps.  A script which needs to only run
once can delete itself after a successful run to prevent a future execution.

# Orderly Shutdown

By default, `docker stop` sends the SIGTERM (and, later, SIGKILL) signal to PID
1 (init) iniside a container.  sysvinit does not act upon this signal.
This will shut down a container, but it will not give your shutdown scripts
the chance to run gracefully.  In many situations, this is fine, but it may
not be so in all.

A workaround is, howerver, readily available, without modifying init.  These
images are configured to perform a graceful shutdown upon receiving `SIGPWR`.

Here, then, is a graceful way to shut down one of these images:

    docker kill -s SIGPWR container
    sleep 10
    docker kill container

Unfortunately, PID 1 cannot be directly killed within the container, so
after the initial application of SIGPWR, the container will essentially be
up but useless.

# Configuration

Althoth the standard and security images run the SMTP and SSH servers,
they do not expose these to the Internet by default.  Both require
site-specific configuration before they are actually useful.

## Email

email is the main thing you'd need to configure.  In the running system,
`dpkg-reconfigure -plow exim4-config` will let you do this.

## SSH

SSH host keys will be generated upon first run of a container, if
they do not already exist.  This implies every instantiation
of a container containing SSH will have a new random host key.
If you want to override this, you can of course supply your own
files in /etc/ssh or make it a volume.

# Source

This is prepared by John Goerzen <jgoerzen@complete.org> and the source
can be found at https://github.com/jgoerzen/docker-debian-base

# Copyright

Docker scripts, etc. are
Copyright (c) 2017 John Goerzen
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:
1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.
3. Neither the name of the University nor the names of its contributors
   may be used to endorse or promote products derived from this software
   without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE AUTHORS AND CONTRIBUTORS ``AS IS'' AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED.  IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
SUCH DAMAGE.

Additional software copyrights as noted.

