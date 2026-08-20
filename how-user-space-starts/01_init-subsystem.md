### How User Space Starts

After kernel is loaded into memory it starts the first user-space process `init`.

**What happens after kernel starts**
1. init system (init is a system and the actual service is `systemd`)
    Start the first user space process, with PID 1, which takes responsibility for bringing entire user space up.
2. low level services like - udevd and syslogd
    These services are started by init subsystem (`systemd`).
    Logging infrastructure like `syslogd`, `systemd-journald` needs to come up early because almost everything that happens during boot can generate diagnostic information.
3. Network configuration
    The kernel already initialized networking subsystem and network drivers during kernel initialization.
    Then the user space processes configures network.
    Example life cycle:
    ```
    kernel
       ↓
    network driver
       ↓
    network interface exists
       ↓
    user-space networking service
       ↓
    configure interfaces
       ↓
    ip address
    routing
    DNS
    etc.
    ```
    The kernel provides networking machinery and user space services configure and manage the network.
    Suppose the computer has:
    ```
    eth0
    ```
    The kernel can know that the network interface exists.
    But user-space configuration can determine:
    ```
    IP Address = 192.168.x.x
    Subnet Mask = 255.255.0.0
    Default Geteway = 192.168.1.1
    DNS = ...
    ```
    Depending on the linux distro, the user-space network processes can involve things like:
    * NetworkManager
    * systemd-networkd
    * DHCP client
    * other networking tools/services
    So Kernel provides network machinery & user-space services configures and manages the network.
    #### Putting Entire Story together
    ```
                    POWER ON
                       │
                       ▼
                BIOS / UEFI
                       │
                       ▼
                    GRUB
                       │
             loads kernel image
                       │
                       ▼
                 LINUX KERNEL
                       │
          initializes kernel subsystems
                       │
                       ▼
              init / systemd (PID 1)
                       │
          ┌────────────┼───────────────┐
          │            │               │
          ▼            ▼               ▼
       udevd         logging        networking
          │            │               │
          └────────────┼───────────────┘
                       │
                       ▼
              other system services
                       │
          ┌────────────┼─────────────┐
          │            │             │
          ▼            ▼             ▼
        cron        printing        sshd
                       │
                       ▼
              login / display manager
                       │
                       ▼
                 user session
                       │
                       ▼
              GUI / shell / apps
                       │
                       ▼
             web server, browser,
             database, etc.
    ```
4. init (systemd) starts mid-level services like - chron, SSH, printing and so on
5. init subsystem starts login prompts, GUIs, and high-level applications, such as web servers.
