*please change stable dockerfiles to download from LTS branch/v1.4*

RELEASE 1.4

Additional features have been put into the 1.4 series of systemctl.py. The
commands of "mask"/"unmask"/"set-default"/"get-default" have been implemented
where the "default" runlevel is only modified by not checked by systemctl.py.
The PID-1 will now assemble all journal logs of services and print them from
the init_loop to stdout so that "docker logs" shows the status logs of all
the services currently running.

Changes include that /var/run/systemd/notify is not the default notify_socket
anymore but instead each service has its own ../notify.A.service channel.
Along with the A.service.lock behaviour one may safely run many systemctl.py
in parallel that will not overlap in getting text. 

The switch to fork/execve in 1.3 had been allowing some process to leave zombies 
or even block the  master systemctl.py which has been resolved now. After all, 
the service.pid parts are completely gone with the MainPID register in the
service.status files now. Checking the (non)active status has been stabilized
from these changes.

The support for usermode containers had been already present in the 1.3 series
but is now tested and documented. The docker-systemctl-images sister project
will show a number of examples of it where the PID-1 of a container is not
run as root but as a specific user - only allowing to start services with the
same user association.

The testsuite has been expanded a lot to support testing usermode containers
which can not be done just by running systemctl --root=path. Additionally the
testing times do increase from testing various pairs of python version and
linux distributions and their versions. Thus --python and --image can be used
to get to the details of compatibility. After all, the coverage is now at 92%.

The question of how to let a PID-1 systemctl.py exit the container has been
put into a new state. When running "systemctl.py init A B" then PID-1 will
end when both A and B have turned to a non-active status. When running a plain
"systemctl.py" then it is the same as "systemctl.py init" which is the same as 
"systemctl.py --init default", all of which will never stop. So this PID-1 is
identical with /sbin/init that keeps it up.

However PID-1 will not run forever in that "init" mode. You can always send a
SIGQUIT, SIGINT, SIGTERM to PID-1. Upon SIGQUIT the PID-1 systemctl.py will wait 
for all processed in the container to have ended. The other signals will make
it leave.

Last not least there are some minor adjustement like adding to the list of
expanded special vars like "%t" runtime. And it was interesting to see how
"postgresql-setup initdb" runs "systemctl show -p Environment" expecting to
get a list of settings in one line. Also when using "User=" then the started
services should also get the default group of that system user.

RELEASE 1.4.2456

There a couple of bugfixes and little enhancements asking for an intermediate
patch release. Be sure to update any 1.4 script to this version soon.

Some of the `%x` special vars were not expanded, and some of the environment
variables like `$HOME` were not present or incorrect. Functionality of the
`mask` command has been improved as well.

The release does also include the extension of the testsuite that was otherwise
intended for RELEASE 1.5 where different versions of common distro images are
included in the nighrun tests. It did uncover a bug in ubuntu `kill` by the
way that may got unnoticed by some application packages so far.

RELEASE 1.4.3000

There are a couple of bugfixes. The most prominent one is the proper support
of drop-in overide configs. This can be very useful in scenarios where one
wants to install a `*.rpm` from an upstream distributor adding some additional
parts only in the case of a docker container image. That has been described
more prominently in [EXTRA-CONFIGS.md].

The general README itself contains an easier introduction with a hint on how
a multi-service container looks like from the inside. That should make some
visual impression on everyone who has already worked with containers so far.

RELEASE 1.4.3424

There are a couple of bugfixes. The most prominent is a missing 'break' in 
the InitLoop (from --init), so that a SIGTERM to PID 1 of a container did 
not initiate a 'halt' shutdown sequence. However that was the intention in
support of a 'docker stop container'.

The ExecStartPre subprocess and the change to WorkingDir did not fail even
when they should. This is fixed now but it may provoke some new errors
downstream. Which is a good thing as the same may happen to that service 
on a systemd controlled machine.

Other than that, testcases were updated to opensuse/leap:15.1. The helper
project 'docker-mirror-packages-repo' does even support more version but
that is not part of the v1.4 anymore.

Since early 2019 there has been an LTS bugfix branch for v1.4 already. The 
difference to the 'master' branch was minimal however. By November 2019
the 'master' branch will be switched to v1.5 which has a couple of 
functional changes. Some projects did not download from LTS branch/v1.4
but directly from master - they should change their processes soon as the 
next rebuild from 'master' may change some behaviour. Specifically, the 
v1.4 generation was implicitly "Restart=no" but the v1.5 generation will 
enable "Restart=always" by default (running from the init loop, i.e. if 
systemctl.py is running on PID 1 or started with --init).

RELEASE 1.4.4147

This version integrates the User=x default groups and any of the specified
SupplementaryGroups when running a service under a different user account.
The 'show <unit>' command does present more values including User/Group.

The python3 type hints in release 1.5.x have revealed some missing checks
for None values that have been added as needed. Those were cherry-picked
into the LTS 1.4 branch. The next release will make python3 the default.

Other than that the testsuite was updated to check2019 with centos 7.7 and
opensuse 15.1 as the defaults. As some distros have stopped shipping a
default '/usr/bin/python' there is also a variant with '/usr/bin/python2'
by now that can be downloaded directly = 'files/docker/systemctl2.py'.
