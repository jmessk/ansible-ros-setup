# Ansible ROS Setup

This repository installs ROS on Ubuntu hosts with Ansible.

- `setup_pc.yaml` targets the `laptop` group.
- `setup_rpi.yaml` targets the `rpi` group.

## Structure

- `group_vars/all.yaml` defines the shared ROS distro.
- `group_vars/laptop.yaml` and `group_vars/rpi.yaml` define which ROS packages each group installs.
- `roles/install_ros` follows the official ROS deb installation guide.

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
```

## Special package installs

Keep `install_ros` for packages that can be installed with:

```bash
apt install ros-<distro>-...
```

If a package needs a source build, a vendor script, or a custom repository, put it in a separate role and add that role to the relevant playbook after `install_ros`.

## Usage

```bash
ansible-playbook -i inventory/hosts.yaml setup_pc.yaml
ansible-playbook -i inventory/hosts.yaml setup_rpi.yaml
```
