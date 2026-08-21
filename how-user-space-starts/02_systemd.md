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

So far we know that after loading the Kernel it starts the first user-space process and i.e. `systemd`, and we know that systemd is a system which creates and manages the system services.

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
There are 2 main directories where `systemd` configuration are stores:
1. /lib/systemd/system or /usr/lib/systemd/system
2. /etc/systemd/system

> **Note: On Ubuntu `/lib` is symlink to `/usr/lib`.**