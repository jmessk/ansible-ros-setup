# Ansible ROS Setup

This repository installs ROS on Ubuntu hosts with Ansible.

- `setup_pc.yaml` targets the `pc` group.
- `setup_rpi.yaml` targets the `rpi` group.

## Structure

- `group_vars/all.yaml` defines the shared ROS distro.
- `group_vars/pc.yaml` and `group_vars/rpi.yaml` define which ROS packages each group installs.
- `roles/pre_setup` applies host-level prerequisites such as the Noble updates repository fix, package upgrades, and chrony.
- `roles/ros_install` follows the official ROS deb installation guide.
- `roles/ros_extra_packages_install` installs extra ROS apt packages from `ros_extra_apt_packages`.
- `roles/tb3_pc_setup` installs Gazebo, Navigation2, Cartographer, and the TurtleBot3 workspace for the remote PC.
- `roles/tb3_sbc_setup` applies the TurtleBot3 SBC-specific workspace and system setup after `ros_install`.
- `roles/tb3_opencr_setup` handles OpenCR-specific dependencies, firmware upload, and udev rules on the SBC.

## Configuration

Set the ROS distro in `group_vars/all.yaml`:

```yaml
ros_distro: jazzy
```

Set the main ROS package, extra ROS apt packages, and whether to install `ros-dev-tools` per group:

```yaml
ros_package: "ros-{{ ros_distro }}-desktop"
ros_extra_apt_packages:
  - "ros-{{ ros_distro }}-rqt-image-view"
ros_install_dev_tools: true
```

For a headless target:

```yaml
ros_package: "ros-{{ ros_distro }}-ros-base"
ros_extra_apt_packages: []
ros_install_dev_tools: false
tb3_opencr_model: burger
```

## Special package installs

Keep `ros_install` for packages that can be installed with:

```bash
apt install ros-<distro>-...
```

Use `ros_extra_packages_install` for additional `ros-<distro>-...` apt packages.

If a package needs a source build, a vendor script, or a custom repository, put it in a separate role and add that role to the relevant playbook after `ros_install`.

`pre_setup` runs before ROS installation and covers host prerequisites that are not specific to ROS packages, including chrony and the Ubuntu Noble package-source fix.

`tb3_pc_setup` covers the remote PC side, including Gazebo Sim, Cartographer, Navigation2, and the TurtleBot3 workspace build.

`tb3_sbc_setup` is the Raspberry Pi side for the TurtleBot3 workspace and SBC-specific system settings.

`tb3_opencr_setup` handles the Raspberry Pi OpenCR path separately: armhf support, firmware upload, and the udev rules needed for the board.

## Usage

```bash
ansible-playbook -i inventory/hosts.yaml setup_pc.yaml
ansible-playbook -i inventory/hosts.yaml setup_rpi.yaml
```
