
### Kernel configuration

If a target platform is known to support buttons, appropriate kernel modules are selected by default.

If a platform is not known to support buttons, you are required to install various kernel modules yourself such as **diag**, **input-gpio-buttons**, **gpio-button-hotplug**, and others.

However, installing various modules will not necessarily yield a successful result.

### Preliminary steps

The first step is to make Hotplug execute scripts in /etc/hotplug.d/button when a button is clicked. Modify /etc/hotplug2.rules — remove '^' before 'button' as follow: 



