### systemd
As we know that `systemd` is the latest implementatino of `init subsystem`.
`systemd` does not only start a service but it keeps track of all the processes(tasks) a service creates, and groups them by `cgroup`.
`systemd` is goal oriented. We can define a goal (`unit`), for some system task.

#### Units and Unit Types
One way that systemd is more ambitous than previous versions of `init` is that it doesn't just operate processes and services; it can also manage filesystem mounts, monitor network connection requests, run timers and more.
Each capability is called *unit type*, and each specific function such as a service is called a *unit*.

* **Service Units**
* **Target Units**
* **Mount Units**
* **Socket Units**


### Booting and Unit Dependency Graphs
How `systemd` decides what to start during boot, and why the dependencies form a graph rather than a simple tree.

So far we know that after loading the Kernel it starts the first user-space process i.e. `systemd`, and we know that systemd is a system which creates and manages the system services.

What `systemd` is tryin to solve is - *What should i activate to bring this machine into its normal operating state?*

It doesn't just randomly start every `.service` file on the machine. Instead it has a particular unit designed as the **default target.**

```
systemctl get-default
```
```
graphical.target
```
For example, simplified:
```
default.target
      │
      ▼
graphical.target
      │
      ├──────────────┐
      ▼              ▼
multi-user.target   display-manager.service
      │
      ├── network
      ├── logging
      ├── ssh
      └── cron
```

**When Linux boots, systemd activates the default target, and that target pulls in a network of dependent units. Because units can depend on multiple other units, the structure is a graph rather than a simple parent-child tree.**

### systemd Configuration
There are 2 main directories where `systemd` configurations are stored:
1. /usr/lib/systemd/system or /lib/systemd/system
2. /etc/systemd/system

> **Note: On Ubuntu `/lib` is a symlink to `/usr/lib`**

So these 2 locations have different use case. The `/usr/lib/systemd/system` stores all the systemd configuration files provided by vendors/packages.
And `/etc/systemd/system` stores systemd service configuration owned by administrators/users.

```
/usr/lib/systemd/system ← vendor/package provided units
/etc/systemd/system     ← administrator/local configuration
```

Both the directories are for different reasons, all the packages we install and ubuntu unit files lives under `/usr/lib/systemd/system`, and we don't want to directly change files under this directory as its managed by ubuntu.
So suppose we change the `ssh.service` file under `/usr/lib/systemd/system` and the `ssh` package is upgraded then our changes will be gone.

Thus we are given this directory `/etc/systemd/system`, where we can create `ssh_override.service.d` kinda file and add its entry in the `override.conf` file so that even when the original package is upgraded our changes will stay.

**So the thumb rule is to never change files under `/usr/lib/systemd/system` (`/lib/systemd/system`) and always make changes and new units under `/etc/systemd/system`.**

**One more important thing is that there are many unit files under `/etc/` which are same as `/usr/lib`, so in most cases they are not the complete files but they are just symlinks to actual units under `/usr/lib`.**

Example:
```
/etc/systemd/system/
├── multi-user.target.wants/
├── graphical.target.wants/
├── timers.target.wants/
├── ssh.service.d/
├── some-custom.service
```

#### .wants directories
We might encounter something like:
```
/etc/systemd/system/multi-user.target.wants
```
so this is not a unit file for `multi-user` but is a directory which stores `symlinks`, like:
```
/etc/systemd/system/multi-user.target.wants/ssh.service
```
Which means `ssh.service` is wanted as part of `multi-user.target`.

#### How does systemd decide which file to use
Suppose there is
```
/usr/lib/systemd/system/foo.service
```
And
```
/etc/systemd/system/foo.service
```
**The `/etc` version takes precedence.**


Suppose there already exists a service `foo.service` in the `/usr/lib/systemd/system` and we want to apply our own modifications to it and run our version of that service, then in that case we don't have to change the service under `/usr/lib`, as this directory is only for vendor/package units and its managed by ubuntu for us.

There is very important behaviour of `systemd` that if there exists `foo.service`, under `/usr/lib/` and also under `/etc` then systemd is going to use the service file from `/etc` directory.

We can check the current systemd confugration search path (including precedence) with this command:
```
systemctl -p UnitPath show
UnitPath=/etc/systemd/system.control /run/systemd/system.control /run/systemd/transient /run/systemd/generator.early /etc/systemd/system /etc/systemd/system.attached /run/systemd/system /run/systemd/system.attached /run/systemd/generator /usr/local/lib/systemd/system /usr/lib/systemd/system /run/systemd/generator.late
```

### Unit Files

**Example 1. dbus-daemon.service**
```
[Unit]
Description=D-Bus something...
Documentation=man:dbus...
Requires=dbus.socket
RefuseManualStart=yes

[Service]
ExecStart=/how/to/start/this/service
ExecReload=/how/to/reload/this/service
```

#### [Unit] section
This section contains General Information about the service.

1. Requires=dbus.socket - It simply means that the `dbus-daemon.service` requires the `dbus.socket`.
      With `Requires`, we define which units are required by the current service. Here in this example the `dbus.socet` unit is required for the `dbus-daemon.service` unit. So systemd will see this section and will try to activate `dbus.socket` service as well.
2. RefuseManualStart=yes - Simply means that this service can't be started with `systemctl start ...`. It might be due to the fact that the unit creator want that the unit should only start based on some conditions and shouldn't be started manually. Thus here `RefuseManualStart` is set to `yes`.
I guess if it's not set then the default is `RefuseManualStart=no`, which means that the unit can be started manually.

#### [Service] section
In the Unit section General information about the unit is declared, but systemd wants to know that how actually is he supposed to start this unit. For example which scripts or binaries to execute. 

So in the Service section of a systemd unit file we give systemd the service-specific instructions on how to start it.

1. This basically tells systemd that When i activate this service, execute this command.
`ExecStart=/usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only`   
2. ExecReload=... - As the name suggest here we write the command or instruction which we want to run when user restarts the service.

**Example 2. ssh.service**

```
[Service]
EnviornmentFile=/etc/sysconfig/sshd
ExecPre=/usr/sbin/sshd-keygen
ExecStart=/usr/sbin/sshd -D $OPTIONS $CRYPTO_POLICY
ExecReload=/bin/kill -HUP $MAINPID
```
***variables ($)***
Here 2 types of $variables are being used:
1. $OPTIONS and $CRYPTO_POLICY - These are normal env variables which are defined in the config file which is set as the value of `EnvironmentFile=/etc/sysconfig/sshd`.
2. $MAINPID - This is special variable, as its not coming from the environment file, but `systemd` actually knows this variable. As we know that systemd tracks all the processes spunned up in the process of activating a service, so when `ssh.serivce` is activated and for e.g. the main process started with `PID=2500` then systemd keeps note of it, and on `ExecReload=/bin/kill -HUP $MAINPID`, systemd knows the value of `$MAINPID`.


***Specifiers (%)*** - These are placeholders and used for different purpose than variable

1. %n - This specifier is a placeholder for name of current service.
2. %H - The Hostname of the machine. Suppose the `hostname` command returns `thinkpad` as the hostname then:
      ```
      %H
      ```
   would represent:
      ```
      thinkpad
      ```

**So Unit files essentially are configuration language through which you tell systemd what a unit is, what it depends on, and how systemd should manage it.**