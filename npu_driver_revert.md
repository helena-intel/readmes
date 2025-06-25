# Un-revert NPU driver

On some AI PC devices, Windows Update can revert the NPU driver to a previous version. At the moment, there is no way to prevent this.

To get the latest driver back:

- Go to Device Manager (type "Device Manager" in the Windows search bar)
- Go to Neural Processors and "unfold" the item 
  - Right click on Intel (R) AI Boost and choose "Uninstall device"
  - Select the button next to "Attempt to uninstall the driver" and press "Uninstall"
- Select the Action menu and choose "Scan for hardware changes". The AI Boost device will be found and added, with the previously installed latest driver. No need to reboot.
