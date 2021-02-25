# Configure Time Synchronization

For an active-active environment with at least two servers in each pair, it's essential that the system clocks match.  Out-of-date clocks between the CAS servers could cause issues with ticket processing, and will make correllating the logs (when doing an investigation - whether it be troubleshooting or incident response) problematic.  The best way to handle this is to ensure the Network Time Protocol (NTP) is in use.

## Determine if NTP is already in use.
RHEL 8 offers chrony for NTP time synchronization.  Older versions of RHEL either had ntpd available, or in use by default, but chrony is what RHEL uses by default now.

If you're looking for more about the differences between chronyd and ntpd, see [Comparison of NTP implementations](https://chrony.tuxfamily.org/comparison.html).  That being said, I've never had issues using whatever the OS default is.


``` bash
    # Check chrony status
    systemctl status chronyd
    # If you are on an older version of RHEL or have otherwise installed NTPD - you can check it as follows:
    systemctl status ntpd
```

If chrony is running, you'll see something like:

<figure>
  <img src="https://paulchauvet.github.io/deploying-cas/images/chrony-service-running.png" alt="Screenshot showing output of systemctl status chronyd where the service is running"/>
</figure>

If the one you check is not installed - you'll see a message like:
```
    Unit chronyd.service could not be found
```

## Install chrony (if needed)

If you don't have chrony installed, you can install it via
``` bash
    dnf install chronyd
```

## Configure chrony
Chrony by default is configured via /etc/chrony.conf.  The default file from RedHat is below, though most options are commented out by default.  The only change I've ever made to it is including our on-prem NTP servers.  If you want to do that, remove the ntp.org servers from the list and replace or supplement them with your own server(s).

``` yaml
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
server 3.rhel.pool.ntp.org iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
#local stratum 10

# Specify file containing keys for NTP authentication.
#keyfile /etc/chrony.keys

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking
```

## Enable and start chronyd
``` bash
    # Set chronyd to start on boot
    systemctl enable chronyd
    # Start chronyd now
    systemctl start chronyd
```

You can check some information about time sync on the system with the command: **chronyc tracking** and **chronyc sources**.

