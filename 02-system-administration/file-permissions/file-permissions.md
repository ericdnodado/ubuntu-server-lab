# Linux File Permissions and Ownership

## 1. Overview

Linux file permissions control how users and groups access files and directories.

As a Systems Administrator, managing permissions is essential for protecting sensitive information, securing application directories, and implementing role-based access control.

This exercise demonstrates Linux permission management using chmod, chown, chgrp, special permission bits, and Access Control Lists (ACLs).

## 2. Objectives

- Understand Linux permission architecture.
- Interpret file permission representations.
- Configure permissions using chmod.
- Manage file and directory ownership.
- Configure group-based access.
- Understand SUID, SGID, and Sticky Bit.
- Implement Access Control Lists (ACLs).
- Validate and troubleshoot permission issues.

## 3. Linux Permission Architecture

Linux permissions are assigned to three categories:

| Category | Description |
|---|---|
| User (u) | File or directory owner |
| Group (g) | Members of the assigned group |
| Others (o) | All other users |

Linux provides three basic permissions:

| Permission | Symbol | Numeric Value | Description |
|---|---|---|---|
| Read | r | 4 | Read file contents or list directory entries |
| Write | w | 2 | Modify files or directory entries |
| Execute | x | 1 | Execute files or access directories |

## 4. Inspect File Permissions

Display files and directories with detailed information:

```bash
ls -l
```

Example output:

```text
-rw-r----- 1 sysadmin linuxadmins 1024 Nov 09 10:00 report.txt
```

Interpretation:

| Field | Value | Meaning |
|---|---|---|
| File type | `-` | Regular file |
| Owner permissions | `rw-` | Read and write |
| Group permissions | `r--` | Read only |
| Others permissions | `---` | No access |
| Owner | `sysadmin` | File owner |
| Group | `linuxadmins` | Assigned group |

Inspect ownership and permission details:

```bash
stat report.txt
```

## 5. Configure Permissions Using chmod

The `chmod` command modifies file and directory permissions.

### 5.1 Numeric Permission Method

Numeric permissions use the values 4, 2, and 1.

Example:

```bash
chmod 755 script.sh
```

Permission breakdown:

| Category | Calculation | Permission |
|---|---|---|
| Owner | 4 + 2 + 1 | rwx |
| Group | 4 + 1 | r-x |
| Others | 4 + 1 | r-x |

### 5.2 Common Permission Configurations

| Permission | Meaning | Typical Use |
|---|---|---|
| 777 | Everyone has full access | Generally avoid |
| 755 | Owner full, others read/execute | Executable directories |
| 750 | Owner full, group read/execute | Restricted application directories |
| 700 | Owner only | Private directories |
| 644 | Owner read/write, others read | Regular files |
| 640 | Owner read/write, group read | Shared configuration files |
| 600 | Owner read/write only | Sensitive files |

### 5.3 Symbolic Permission Method

Add execute permission for the owner:

```bash
chmod u+x script.sh
```

Remove write permission for others:

```bash
chmod o-w report.txt
```

Give the group read and write permissions:

```bash
chmod g+rw report.txt
```

Remove all permissions for others:

```bash
chmod o-rwx report.txt
```

## 6. Manage File Ownership Using chown

The `chown` command changes the owner and optionally the group.

### 6.1 Change File Owner

```bash
sudo chown sysadmin report.txt
```

### 6.2 Change Owner and Group

```bash
sudo chown sysadmin:linuxadmins report.txt
```

### 6.3 Change Group Ownership

```bash
sudo chgrp linuxadmins report.txt
```

### 6.4 Recursive Ownership Changes

```bash
sudo chown -R sysadmin:linuxadmins /srv/linuxadmins
```

Recursive changes apply to the specified directory and its contents.

**Warning:** Always verify the target directory before performing recursive ownership changes.

## 7. Directory Permissions

Directory permissions behave differently from regular file permissions.

| Permission | Directory Behavior |
|---|---|
| Read | List directory entries |
| Write | Create, delete, or rename directory entries, subject to other restrictions |
| Execute | Traverse or access directory entries when permitted |

Example:

Create an administrative directory:

```bash
sudo mkdir -p /srv/linuxadmins
```

Set ownership:

```bash
sudo chown root:linuxadmins /srv/linuxadmins
```

Set permissions:

```bash
sudo chmod 770 /srv/linuxadmins
```

Verify:

```bash
ls -ld /srv/linuxadmins
```

## 8. Special Permission Bits

Linux supports three special permission bits.

### 8.1 SUID (Set User ID)

SUID allows an executable file to run with the effective user ID of its owner.

Example:

```bash
ls -l /usr/bin/passwd
```

The `s` permission indicator in the owner's execute position identifies SUID.

Set SUID:

```bash
sudo chmod u+s application
```

Remove SUID:

```bash
sudo chmod u-s application
```

### 8.2 SGID (Set Group ID)

SGID on an executable file causes it to run with the effective group ID of the file.

SGID on a directory causes newly created files and subdirectories to inherit the directory's group.

Example:

```bash
sudo chmod 2770 /srv/linuxadmins
```

Verify:

```bash
ls -ld /srv/linuxadmins
```

Expected permission representation:

```text
drwxrws---
```

### 8.3 Sticky Bit

The sticky bit on a directory restricts deletion or renaming of entries to the entry owner, directory owner, or a privileged user.

Example:

```bash
sudo chmod 1777 /srv/shared
```

Verify:

```bash
ls -ld /srv/shared
```

A common example is:

```text
drwxrwxrwt
```

The `/tmp` directory commonly uses the sticky bit.

## 9. Access Control Lists (ACL)

ACLs provide additional permission control beyond standard Linux owner, group, and others permissions.

They allow administrators to grant specific users or groups access to individual files and directories.

### 9.1 Install ACL Utilities

```bash
sudo apt update
sudo apt install acl -y
```

### 9.2 Assign User ACL

Grant a specific user read and write access:

```bash
sudo setfacl -m u:sysadmin:rw report.txt
```

### 9.3 Assign Group ACL

Grant a group read access:

```bash
sudo setfacl -m g:linuxadmins:r report.txt
```

### 9.4 Display ACL Configuration

```bash
getfacl report.txt
```

### 9.5 Remove a Specific ACL Entry

```bash
sudo setfacl -x u:sysadmin report.txt
```

### 9.6 Remove All Extended ACL Entries

```bash
sudo setfacl -b report.txt
```

### 9.7 Configure Default ACL

Default ACLs allow newly created objects within a directory to inherit specified ACL entries.

```bash
sudo setfacl -m d:g:linuxadmins:rwx /srv/linuxadmins
```

Inspect the directory ACL:

```bash
getfacl /srv/linuxadmins
```

**Note:** Default ACL inheritance is subject to the permissions requested when new objects are created.

## 10. Practical Enterprise Scenario

### Scenario: Shared IT Department Directory

An organization requires a shared directory for IT administrators.

Requirements:

- Directory: `/srv/it-department`
- Directory owner: `root`
- Group: `linuxadmins`
- Group members: Authorized IT administrators
- Owner permissions: Full access
- Group permissions: Full access
- Others: No access
- New files should inherit the `linuxadmins` group.

### Step 1: Create the Group

```bash
sudo groupadd linuxadmins
```

### Step 2: Create the Directory

```bash
sudo mkdir -p /srv/it-department
```

### Step 3: Configure Ownership

```bash
sudo chown root:linuxadmins /srv/it-department
```

### Step 4: Configure SGID and Permissions

```bash
sudo chmod 2770 /srv/it-department
```

### Step 5: Add Authorized Users

```bash
sudo usermod -aG linuxadmins sysadmin
```

Existing sessions may need to be restarted for supplementary group membership to take effect.

### Step 6: Verify Configuration

```bash
ls -ld /srv/it-department
```

Expected permission pattern:

```text
drwxrws---
```

### Step 7: Test Access

Switch to the authorized account:

```bash
su - sysadmin
```

Create a test file:

```bash
touch /srv/it-department/test-file.txt
```

Inspect the file:

```bash
ls -l /srv/it-department/
```

The new file should inherit the `linuxadmins` group.

### Step 8: Validate Unauthorized Access

Test access using an account that is not a member of `linuxadmins`.

```bash
ls /srv/it-department/
```

The unauthorized user should receive a permission-denied error.

## 11. Permission Troubleshooting

| Problem | Investigation |
|---|---|
| Permission denied | Check `ls -l` and `stat` |
| Wrong ownership | Check `chown` and `chgrp` |
| User cannot access directory | Check execute permission on parent directories |
| Group access not working | Check `id` and supplementary groups |
| ACL access unexpected | Inspect `getfacl` |
| New files have incorrect group | Check SGID on the parent directory |
| Application cannot read file | Inspect file permissions and service account identity |

Useful commands:

```bash
namei -l /srv/linuxadmins/test-file.txt
```

```bash
getfacl /srv/linuxadmins
```

```bash
id sysadmin
```

## 12. Security Best Practices

- Follow the principle of least privilege.
- Avoid unnecessary `777` permissions.
- Restrict access to sensitive configuration files.
- Use groups for shared administrative access.
- Apply ACLs only when standard permissions are insufficient.
- Review special permission bits regularly.
- Avoid unrestricted recursive permission changes.
- Validate permissions after configuration changes.

## 13. Lab Validation Checklist

| Test | Status |
|---|---|
| Inspect file permissions | Pending |
| Configure numeric permissions | Pending |
| Configure symbolic permissions | Pending |
| Change file ownership | Pending |
| Configure directory permissions | Pending |
| Configure SGID | Pending |
| Configure sticky bit | Pending |
| Configure ACL | Pending |
| Test authorized access | Pending |
| Test unauthorized access | Pending |

## 14. Conclusion

This exercise demonstrates Linux file permission management, ownership administration, special permission bits, and ACL implementation.

These capabilities are essential for securing shared resources, controlling administrative access, and maintaining enterprise Linux environments.

**Project:** Ubuntu Server Administration Lab  
**Module:** Linux System Administration  
**Exercise:** File Permissions and Ownership  
**Platform:** Ubuntu Server
