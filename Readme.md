# What's in this guide ?

Here I make notes and check boxes for my installing journey of QEMU/KVM
I am using this guide on my machine with :

- Arch-Linux (EndeavourOS) 6.8.8-arch1-1
- GDM
- x11
  
PC Specs are :

- | ASUSTeK COMPUTER INC. ASUS TUF Dash F15 FX516PM |
- | CPU 11th Gen Intel® Core™ i7-11370H @ 3.30GHz (4)|
- | dGPU NVIDIA GeForce RTX 3060 Laptop GPU |

----------------------------------------------------------------

## To DO

- Qemu/KVM install/config :
  - [x] Installing packages :
    - `yay -Suy && yay qemu libvirt edk2-ovmf virt-manager virt-viewer swtpm iptables-nft dnsmasq`
  - [x] Enable libvirt using systemctl
    - `sudo systemctl enable --now libvirtd.service`
    - check libvirt status :
    - `systemctl status libvirtd.service`
  - [x] Enable/Start the default network for VM's
    - `sudo virsh net-autostart default`
    - `sudo virsh net-start default`
  - [x] Add user to libvirt group
    - `sudo usermod -a -G libvirt-qemu ($yourUserName) && reboot`
  - [x] Installing a polkit if you don't have one
    - `yay polkit-gnome`
  - [x] Run the polkit in a different terminal or from your startup
    - `/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1`
  - [x] Download Virtio-Win drivers from 
    - [https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md](https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md)

- GPU Passthrough :
  - [x] Download Virtio-