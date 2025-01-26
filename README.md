# GlusterFS High Availability Setup

This guide walks through setting up a shared volume with GlusterFS across a network to ensure high availability. The configuration involves three storage nodes (`fs1`, `fs2`, and `fs3`) and one worker node (`worker`). Each storage node hosts a replica of the Odoo cache, while the worker hosts the Odoo service (either source-installed or Dockerized).

## Prerequisites
- Machines:
  - `fs1` (192.168.6.54)
  - `fs2` (192.168.6.52)
  - `fs3` (192.168.6.53)
  - `worker` (192.168.6.50)
- Hostnames should resolve correctly (e.g., via `/etc/hosts` or DNS).

---

## Step 1: Install and Start GlusterFS
### On all nodes (`fs1`, `fs2`, `fs3`, and `worker`):

```bash
sudo apt-get update
sudo apt-get install glusterfs-server -y
sudo systemctl enable glusterd
sudo systemctl start glusterd
```
### Ensure glusterfs on all nodes share same version

---

## Step 2: Configure GlusterFS Cluster
### On `fs1`:
Add `fs2` and `fs3` to the trusted pool:
```bash
gluster peer probe fs2
gluster peer probe fs3
```
You can verify the peer status:
```bash
gluster peer status
```

---

## Step 3: Create and Start the Volume
### On `fs1`:
Create a GlusterFS volume with 3 replicas:
```bash
gluster volume create <volume_name> replica 3 transport tcp \
  fs1:/path/to/data \
  fs2:/path/to/data \
  fs3:/path/to/data
```
Start the volume:
```bash
gluster volume start <volume_name>
```
Verify the volume status:
```bash
gluster volume status
```

---

## Step 4: Mount the Volume on Worker
### On `worker`:
1. Install the GlusterFS client:
   ```bash
   sudo apt-get install glusterfs-client -y
   ```
2. Mount the GlusterFS volume:
   ```bash
   sudo mount -t glusterfs fs1:/<volume_name> /path/to/mount
   ```
3. To ensure the volume mounts automatically on reboot, edit `/etc/fstab`:
   ```bash
   sudo nano /etc/fstab
   ```
   Add the following line:
   ```
   fs1:/<volume_name> /path/to/mount glusterfs defaults,_netdev 0 0
   ```

---

## Step 5: Set Permissions
### On `worker`:
Set the necessary permissions for the mount point:
```bash
sudo chown -R <user>:<user> /path/to/mount
sudo chmod 777 /path/to/mount
```

---

## Step 6: Test High Availability
1. Shut down one of the storage nodes (`fs1`, `fs2`, or `fs3`).
2. Verify that the volume is still accessible on the worker node:
   ```bash
   ls /path/to/mount
   ```

---

## Notes
- Replace `<volume_name>` with your desired volume name.
- Replace `/path/to/data` with the actual directory path on the storage nodes.
- Replace `/path/to/mount` with the actual mount directory on the worker node.
- Replace `<user>` with the appropriate username.

By following this guide, you will have a highly available GlusterFS volume that remains accessible even if one storage node goes down.

