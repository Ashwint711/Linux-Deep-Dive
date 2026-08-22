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
**The `/etc` version takes precedense.**


Suppose there already exists a service `foo.service` in the `/usr/lib/systemd/system` and we want to apply our own modifications to it and run our version of that service, then in that case we don't have to change the service under `/usr/lib`, as this directory is only for vendor/package units and its managed by ubuntu for us.

There is very important behaviour of `systemd` that if there exists `foo.service`, under `/usr/lib/` and also under `/etc` then systemd is going to use the service file from `/etc` directory.