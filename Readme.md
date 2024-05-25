# QEMU Guide

# **What's in this guide ?**

Here I make notes and check boxes for my installing journey of QEMU/KVM I am using this guide on my machine with :

- Arch-Linux (EndeavourOS) 6.8.8-arch1-1
- Gnome 46.1 DM
- Mutter WM
- X11

My Laptop Specs are :

- | ASUSTeK COMPUTER INC. ASUS TUF Dash F15 FX516PM |
- | CPU 11th Gen Intel Core™ i7-11370H @ 3.30GHz (4)|
- | dGPU NVIDIA GeForce RTX 3060 Laptop GPU Mobile / Max-Q |
- | iGPU Intel TigerLake-LP GT2 [Iris Xe Graphics] |

**Useful wikis :**



[https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

[https://looking-glass.io/docs/B6/install/](https://looking-glass.io/docs/B6/install/)

---

### **Note** 

If you are using asusctl and supergfxctl for Easy Graphics Switching then you better head to this guide article

[https://asus-linux.org/guides/vfio-guide/](https://asus-linux.org/guides/vfio-guide/)

---

# **To DO**

- **Qemu/KVM install/config :**
  - [ ]  Installing packages :
    - `yay -Suy && yay qemu libvirt edk2-ovmf virt-manager virt-viewer swtpm iptables-nft dnsmasq`
  - [ ]  Enable libvirt using systemctl
    - `sudo systemctl enable --now libvirtd.service`
    - check libvirt status :
    - `systemctl status libvirtd.service`
  - [ ]  Enable/Start the default network for VM's
    - `sudo virsh net-autostart default`
    - `sudo virsh net-start default`
  - [ ]  Add user to libvirt group
    - `sudo usermod -a -G libvirt-qemu ($yourUserName) && reboot`
  - [ ]  Installing a polkit if you don't have one
    - `yay polkit-gnome`
  - [ ]  Run the polkit in a different terminal or from your startup
    - `/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1`
  - [ ]  Download Virtio-Win drivers from
    - [https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md)

---

- **QEMU/KVM configs :**
  - Firmware must be : UEFI
  - Change Network interface device model to “**virtio**”

        ```xml
        <interface type="network">
          ....
          <model type="virtio"/>
          ....
        </interface>
        
        ```

  - Change the bus type to ‘**virtio**’ ****for mouse and keyboard :

        ```xml
        <input type="keyboard" bus="virtio">
        .....
        </input>
        /// and for mouse its gonna be like this :
        <input type="mouse" bus="virtio">
        .....
        </input>
        ```

---

- **GPU Passthrough :**
    1. Preparing :
        - [x]  Find GPU ID with its Audio Device
            - `lspci -nnk` or you can just do  `lspci -nnk | grep 'NVIDIA*'`
            - for me it was : gpu = `10de:2520` |  audio = `10de:228e`

        [PCI passthrough Arch wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF)

        <aside>
        💡 **Tip:** You can do steps 2— with **[gpu-passthrough-manager](https://aur.archlinux.org/packages/gpu-passthrough-manager/)** AUR. This is a simple graphical interface that lets you choose what graphics drivers you want to load on your system. This can also set up IOMMU and adds VFIO modules, but is limited to GRUB only. Select the graphics and audio devices you want to passthrough and then select LOAD VFIO. Reboot when prompted and then continue to step 4.

        </aside>

    2. **Setting up IOMMU**
        - [x]  Your CPU must support hardware virtualization (for kvm) and IOMMU (for the passthrough itself).
            - you can check for cpu virtualization support by : `lscpu | grep 'Virt*'`

            <aside>
            💡 **Note:**

            - IOMMU is a generic name for Intel VT-d and AMD-Vi.
            - VT-d stands for *Intel Virtualization Technology for Directed I/O* and should not be confused with VT-x *Intel Virtualization Technology*. VT-x allows one hardware platform to function as multiple “virtual” platforms while VT-d improves security and reliability of the systems and also improves performance of I/O devices in virtualized environments.
            </aside>

        - [ ]  Manually enable IOMMU support by setting the correct **[kernel parameter](https://wiki.archlinux.org/title/Kernel_parameter)** depending on the type of CPU in use:
            - For Intel CPUs (VT-d) set `intel_iommu=on`. Since the kernel config option CONFIG_INTEL_IOMMU_DEFAULT_ON is not set in **[linux](https://archlinux.org/packages/?name=linux)**.
            - For AMD CPUs (AMD-Vi), it is on if kernel detects IOMMU hardware support from BIOS.
        - [ ]  Append the `iommu=pt` parameter
        - After rebooting, check **[dmesg](https://wiki.archlinux.org/title/Dmesg)** to confirm that IOMMU has been correctly enabled:

            ```bash
            # dmesg | grep -i -e DMAR -e IOMMU
            [    0.000000] ACPI: DMAR 0x00000000BDCB1CB0 0000B8 (v01 INTEL  BDW      00000001 INTL 00000001)
            [    0.000000] Intel-IOMMU: enabled
            [    0.028879] dmar: IOMMU 0: reg_base_addr fed90000 ver 1:0 cap c0000020660462 ecap f0101a
            [    0.028883] dmar: IOMMU 1: reg_base_addr fed91000 ver 1:0 cap d2008c20660462 ecap f010da
            [    0.028950] IOAPIC id 8 under DRHD base  0xfed91000 IOMMU 1
            [    0.536212] DMAR: No ATSR found
            [    0.536229] IOMMU 0 0xfed90000: using Queued invalidation
            [    0.536230] IOMMU 1 0xfed91000: using Queued invalidation
            [    0.536231] IOMMU: Setting RMRR:
            [    0.536241] IOMMU: Setting identity map for device 0000:00:02.0 [0xbf000000 - 0xcf1fffff]
            [    0.537490] IOMMU: Setting identity map for device 0000:00:14.0 [0xbdea8000 - 0xbdeb6fff]
            [    0.537512] IOMMU: Setting identity map for device 0000:00:1a.0 [0xbdea8000 - 0xbdeb6fff]
            [    0.537530] IOMMU: Setting identity map for device 0000:00:1d.0 [0xbdea8000 - 0xbdeb6fff]
            [    0.537543] IOMMU: Prepare 0-16MiB unity mapping for LPC
            [    0.537549] IOMMU: Setting identity map for device 0000:00:1f.0 [0x0 - 0xffffff]
            [    2.182790] [drm] DMAR active, disabling use of stolen memory
            ```

        - [ ]  **Ensuring that the groups are valid :**

            The following script should allow you to see how your various PCI devices are mapped to IOMMU groups. If it does not return anything, you either have not enabled IOMMU support properly or your hardware does not support it.

            ```bash
            #!/bin/bash
            shopt -s nullglob
            for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
                echo "IOMMU Group ${g##*/}:"
                for d in $g/devices/*; do
                    echo -e "\t$(lspci -nns ${d##*/})"
                done;
            done;
            
            ```

            - Example output:

                ```bash
                IOMMU Group 1:
                 00:01.0 PCI bridge: Intel Corporation Xeon E3-1200 v2/3rd Gen Core processor PCI Express Root Port [8086:0151] (rev 09)
                IOMMU Group 2:
                 00:14.0 USB controller: Intel Corporation 7 Series/C210 Series Chipset Family USB xHCI Host Controller [8086:0e31] (rev 04)
                IOMMU Group 4:
                 00:1a.0 USB controller: Intel Corporation 7 Series/C210 Series Chipset Family USB Enhanced Host Controller #2 [8086:0e2d] (rev 04)
                IOMMU Group 10:
                 00:1d.0 USB controller: Intel Corporation 7 Series/C210 Series Chipset Family USB Enhanced Host Controller #1 [8086:0e26] (rev 04)
                IOMMU Group 13:
                 06:00.0 VGA compatible controller: NVIDIA Corporation GM204 [GeForce GTX 970] [10de:13c2] (rev a1)
                 06:00.1 Audio device: NVIDIA Corporation GM204 High Definition Audio Controller [10de:0fbb] (rev a1)
                
                ```

    3. Isolating the GPU :

        <aside>
        ⚠️ **Warning:** Once you reboot after this procedure, whatever GPU you have configured will no longer be usable on the host until you reverse the manipulation. Make sure the GPU you intend to use on the host is properly configured before doing this - your motherboard should be set to display using the host GPU.

        </aside>

        - [ ]  Edit grub and add `intel_iommu=on` and `vfio-pci.ids=10de:2520,10de:228e` to `GRUB_CMDLINE_LINUX_DEFAULT`  
        |||| Dont forget to change your ids
        save it then update grub with `sudo grub-mkconfig -o /boot/grub/grub.cfg`  
        then reboot your pc
        - [ ]  Make a file called `‘vfio.conf’` in  this dir `‘/etc/modprobe.d/’`
        or just use this command `sudo vim /etc/modprobe.d/vfio.conf`
        write in that file :  [don’t forget to change ids to your ids]

        ```bash
        options vfio-pci ids=10de:2520,10de:228e
        softdep nvidia pre: vfio-pci
        ```

        - [ ]  Edit the `'mkinitcpio.conf'` file in `/etc/mkinitcpio.conf` and add `‘vfio_pci vfio vfio_iommu_type1 ’` modules to ‘**MODULES’  
        - Note** you dont have to add **`vfio_virqfd`** since its already now built into the kernel instead of being a module.
        - [ ]  rebuild the initial RAM filesystem `sudo mkinitcpio -P`
        - [ ]  Reboot
        - [ ]  Add GPU & Audio device PCI to virt-manager

---

- **Looking Glass :**
    1. Adding IVSHMEM Device to virtual machines
        1. adding this to the vm xml

        ```xml
        ...
        <devices>
            ...
          <shmem name='looking-glass'>
            <model type='ivshmem-plain'/>
            <size unit='M'>32</size>
          </shmem>
        </devices>
        ...
        ```

        You should replace 32 with your own calculated value based on what resolution you are going to pass through. It can be calculated like this:

        ```
        width x height x 4 x 2 = total bytes
        total bytes / 1024 / 1024 = total mebibytes + 10
        
        ```

        For example, in case of 1920x1080

        ```
        1920 x 1080 x 4 x 2 = 16,588,800 bytes
        16,588,800 / 1024 / 1024 = 15.82 MiB + 10 = 25.82
        
        ```

        The result must be **rounded up** to the nearest power of two, and since 25.82 is bigger than 16 we should choose 32.

        see the [wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Adding_IVSHMEM_Device_to_virtual_machines) for more

    2. Create a configuration file to create the shared memory file on boot

        `/etc/tmpfiles.d/10-looking-glass.conf`

        write in that conf file : change [user] to ur username

        ```xml
        f /dev/shm/looking-glass 0660 ~~user~~ kvm -
        ```

    3. Ask systemd-tmpfiles to create the shared memory file now without waiting to next boot

        `sudo systemd-tmpfiles --create /etc/tmpfiles.d/10-looking-glass.conf`

    4. **Installing the IVSHMEM Host to Windows guest**
        1. Download the signed driver **[from Red Hat](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/upstream-virtio/)**.
        2. Currently Windows would not notify users about a new IVSHMEM device, it would silently install a dummy driver. To actually enable the device you have to go into device manager and update the driver for the device under the "System Devices" node for **"PCI standard RAM Controller"**.
        3. Download a matching **[looking-glass-host](https://looking-glass.io/downloads)** package that matches the client you will install from AUR, and install it on your guest.

    5. **Setting up the null video device**
        1. If you would like to use Spice to give you keyboard and mouse input along with clipboard sync support, make sure you have a `<graphics type='spice'>` device, then:
            - Find your `<video>` device, and set `<model type='none'/>`
            - If you cannot find it, make sure you have a `<graphics>` device, save and edit again

    6. **If you don’t have `Scrlk` key in ur keyboard**
        1. Looking glass uses scrlk key as default key for capture key
        2. if you wanna change it do

            `looking-glass-client -m 88` here i choose 88 which is `F12` key in linux

            see this helpful github file for more keys [https://gist.github.com/rickyzhang82/8581a762c9f9fc6ddb8390872552c250](https://gist.github.com/rickyzhang82/8581a762c9f9fc6ddb8390872552c250)

        3. Here is my looking glass conf (for now i still didt change much in it)

            ```xml
            shmFile=/dev/shm/looking-glass
            
            [win]
            title=looking-glass-client
            size=1920x1080
            keepAspect=yes
            fullScreen=yes
            uiFont=pango:Iosevka
            uiSize=14
            #maximize=yes
            showFPS=yes
            
            #[egl]
            #vsync=yes
            #multisample=yes
            #scale=0
            
            #[wayland]
            #warpSupport=yes
            #fractionScale=yes
            
            [input]
            #grabKeyboardOnFocus=yes
            #autoCapture=yes
            escapeKey=88
            
            #[spice]
            #enable=no
            ```

        4. To start using this conf file, save it anywhere you want
        for me its `~/looking-glass.ini`
        5. then you start looking glass by : `looking-glass-client -C looking-glass.ini`
