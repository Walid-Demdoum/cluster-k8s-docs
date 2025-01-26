# VirtualBox Commands and Configuration

## Installing VirtualBox
```bash
sudo apt install virtualbox # Install VirtualBox
```

## Managing Virtual Machines with VBoxManage
- **Import a preconfigured VM:**
  ```bash
  vboxmanage import /path/to/virtual_machine.ova
  ```
- **List all installed VMs:**
  ```bash
  vboxmanage list vms
  ```
- **List all running VMs:**
  ```bash
  vboxmanage list runningvms
  ```
- **Start a VM (headless):**
  ```bash
  vboxmanage startvm <vm_name_or_uuid> --type headless
  ```
- **Power off a VM:**
  ```bash
  vboxmanage controlvm <vm_name_or_uuid> poweroff
  ```
- **Delete an installed VM:**
  ```bash
  vboxmanage unregistervm <vm_name_or_uuid> --delete
  ```
- **Enable nested virtualization:**
  ```bash
  VBoxManage modifyvm --nested-hw-virt on
  ```

## Download VirtualBox Extension Pack
- [Download Extension Pack](https://download.virtualbox.org/virtualbox/6.1.44/Oracle_VM_VirtualBox_Extension_Pack-6.1.44-156814.vbox-extpack)

# Configuring and Using a New Disk in Debian

## 1. Verify Disk Recognition
1. Boot your Debian VM.
2. Open a terminal window.
3. Run the command to list all attached disks:
   ```bash
   fdisk -l
   ```
4. Identify the new 40GB disk by its size (usually labeled as `/dev/sdX`, where `X` is a letter).

## 2. Partition the New Disk
1. Use `fdisk` to partition the new disk:
   ```bash
   fdisk /dev/sdX
   ```
2. Inside `fdisk`, follow these steps:
   - Type `n` to create a new partition.
   - Select `p` for a primary partition.
   - Choose partition number `1` (or another if required).
   - Accept the default values for first and last sector to use the entire disk space.
   - Type `t` to change the partition type.
   - Enter `83` for a Linux partition.
   - Type `w` to write the changes.

## 3. Format the Partition
1. Exit `fdisk` by typing `q`.
2. Format the partition with the ext4 filesystem:
   ```bash
   mkfs.ext4 /dev/sdX1
   ```

## 4. Mount the New Partition
1. Create a mount point directory:
   ```bash
   sudo mkdir /mnt/postgres_data
   ```
2. Mount the partition:
   ```bash
   sudo mount /dev/sdX1 /mnt/postgres_data
   ```

## 5. Update `/etc/fstab` (Optional)
1. Open the file:
   ```bash
   sudo nano /etc/fstab
   ```
2. Add the following line at the end (replace `/dev/sdX1` and `/mnt/postgres_data` with your values):
   ```bash
   /dev/sdX1  /mnt/postgres_data  ext4  defaults  0  2
   ```
3. Save and exit (`Ctrl+O`, then `Ctrl+X`).

## 6. Move PostgreSQL Data
1. Stop PostgreSQL:
   ```bash
   sudo systemctl stop postgresql
   ```
2. Move the PostgreSQL data directory:
   ```bash
   sudo mv /var/lib/postgres/data /mnt/postgres_data/
   ```
3. Create a symbolic link:
   ```bash
   sudo ln -s /mnt/postgres_data/data /var/lib/postgres/
   ```

## 7. Verify Permissions
1. Ensure the PostgreSQL user and group own the data directory:
   ```bash
   sudo chown -R postgres:postgres /mnt/postgres_data
   ```

## 8. Restart PostgreSQL
1. Start PostgreSQL:
   ```bash
   sudo systemctl start postgresql
   ```

# Additional Considerations

## Data Integrity
- Back up your database before making changes to avoid data loss or corruption.

## PostgreSQL Configuration
- Adjust configuration files if they reference the data directory location.

## Future Growth
- To add more space, repeat these steps to integrate another disk.

By carefully following these instructions, you can successfully migrate `/var/lib/postgres` to a new disk on your Debian VM.

