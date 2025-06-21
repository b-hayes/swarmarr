# MergerFS Array Setup Guide

MergerFS combines multiple mount points into a single shared mount point, allowing you to spread your media across multiple drives without your apps needing to know about it.

## Install MergerFS
If your on ubuntu you can install mergerfs with:
```shell
sudo apt install mergerfs
```

## Mount the individual drives first.
Disks GUI - it's way easier:

Open Disks app
Click on each drive
Gear icon → "Edit Mount Options"
Turn off "User Session Defaults"
Set your mount point (/mnt/drive00, etc.)
Check "Mount at system startup"
Done

The GUI will handle all the configuration and create the folder for you.

## Create Array Mount Point

Create the directory where your merged filesystem will be mounted:

```bash
sudo mkdir -p /array
sudo chown $USER:$USER /array
sudo chmod 755 /array
```

## Edit fstab

Open the fstab file for editing:

```bash
sudo nano /etc/fstab
```

## Add MergerFS Entry

Add this line to your fstab adjust the mount points as needed:

```text
/mnt/drive00:/mnt/drive01 /array mergerfs cache.files=off,category.create=pfrd,func.getattr=newest,dropcacheonclose=false,allow_other 0 0
```

## Test the Mount

First reload systemd to apply fstab changes:

```shell
sudo systemctl daemon-reload
```
Then test the mount:
```bash
sudo mount /array
```

## Check Permissions

Verify you have proper access:

```bash
ls -la /array
```

## Fix Permissions (if needed)

If you don't have access, fix the ownership:

```bash
sudo chown $USER:$USER /array
```

## Notes

- Individual drives (`/mnt/drive00`, `/mnt/drive01`) are handled by the Disks app
- This setup will automatically mount the merged filesystem on boot
- The array will provide a unified view of both drives with proper user permissions


## Documentation
- [MergerFS Documentation](https://github.com/trapexit/mergerfs)