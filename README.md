## 🧍 User Management in Linux: In-Depth Guide

---

### 1. User Accounts

#### Types of Users
- **Root User (UID 0)**: Has unrestricted access to all system files and commands.
- **System Users (UID < 1000)**:
  - Created during OS installation or package installation.
  - Used by system services (e.g., `www-data`, `mysql`).
- **Regular Users (UID >= 1000)**:
  - Human users created by the system administrator.

# 📘 Essential Files in Linux User Management

Linux user management relies on several critical files that store account details, passwords, default settings, and templates. This document provides an in-depth explanation of each essential file, including historical context, structure, use cases, and security.

---

## 📂 `/etc/passwd`

The `/etc/passwd` file is one of the oldest and most foundational configuration files in Unix-like systems. Initially, it held **all user information**, including **unencrypted passwords**. For security reasons, password storage was later moved to `/etc/shadow`, leaving `/etc/passwd` to store only public user metadata.

### Historical Purpose
Originally, `/etc/passwd` held:
- Username
- Encrypted password
- User and group IDs
- GECOS field
- Home directory
- Login shell

### Format:
Each line is one user record:
```
username:password:UID:GID:comment:home_directory:login_shell
```

### Example:
```
john:x:1001:1001:John Doe:/home/john:/bin/bash
```

### Field Explanation:
- `username`: Login name (e.g., `john`)
- `password`: Usually `x` (placeholder), real password is in `/etc/shadow`
- `UID`: User ID; `0` = root, `>=1000` = regular users
- `GID`: Group ID; maps to `/etc/group`
- `comment`: Full name or description (GECOS field)
- `home_directory`: Default directory after login
- `login_shell`: Default shell, e.g., `/bin/bash`

### Access Control:
```bash
-rw-r--r-- 1 root root 1234 Apr 10 12:34 /etc/passwd
```
- Readable by everyone
- Writable only by root

This universal readability allows commands like `ls -l` to resolve UID to username.

### Modern Relevance
Despite moving password hashes out, `/etc/passwd` is still used by:
- `login`, `sshd`, `su`, and other authentication programs
- `ls`, `ps`, and system utilities

It acts as a bridge between the system and human-readable user data.

---

## 🔐 `/etc/shadow`

### Purpose
Introduced to **enhance security**, this file stores **encrypted user passwords** and **account aging information**. Unlike `/etc/passwd`, it is **readable only by root**, preventing unprivileged users from accessing password hashes.

### Format:
Each line contains 9 colon-separated fields:
```
username:encrypted_password:last_change:min:max:warn:inactive:expire:reserved
```

### Key Details:
- `username`: Matches entry in `/etc/passwd`
- `encrypted_password`: Typically hashed with SHA-512 (starts with `$6$`)
- `last_change`: Days since epoch (1970-01-01)
- `min`, `max`: Control when password changes are allowed or required
- `warn`: Days to warn user before expiry
- `inactive`, `expire`: Post-expiration policies

### Security:
```bash
-r-------- 1 root root /etc/shadow
```
- Only root or PAM-aware utilities like `passwd` can access this
- Use `vipw -s` to edit safely

---

## 📋 `/etc/group`

### Purpose
Stores group-related metadata. Helps define **secondary group memberships**, which allow users to share access to files or devices.

### Format:
```
group_name:password:GID:members
```

### Example:
```
wheel:x:10:john,alice
```

- `group_name`: e.g., `wheel`, `sudo`
- `password`: Often unused (`x`), used with `newgrp`
- `GID`: Group identifier
- `members`: Comma-separated list of users

### Practical Use:
Used by utilities like:
- `groups`, `id`, `newgrp`, `chgrp`, `sudo`

### Access:
```bash
-rw-r--r-- 1 root root /etc/group
```

---

## 🔒 `/etc/gshadow`

### Purpose
Acts as the **shadow version of `/etc/group`**, holding secure group passwords and administrative privileges.

### Format:
```
group_name:password:admin_list:member_list
```

- `password`: Encrypted group password or `!` if not used
- `admin_list`: Users allowed to modify the group
- `member_list`: Normal members

### Security:
```bash
-r-------- 1 root root /etc/gshadow
```
Must be edited with `vigr -s` to maintain system integrity.

---

## 📈 `/etc/login.defs`

### Purpose
Contains default **user account creation policies**. Used primarily by tools like `useradd`, `passwd`, and PAM.

### Examples of Settings:
- `UID_MIN`, `UID_MAX`: Define normal user UID range
- `PASS_MAX_DAYS`, `PASS_MIN_DAYS`: Password aging
- `ENCRYPT_METHOD`: Password hash algorithm (e.g., SHA512)
- `CREATE_HOME`: If true, `useradd` will create a home directory

Changes here affect **future** users, not existing ones.

---

## 🌐 `/etc/skel/`

### Purpose
Holds **template configuration files** copied into every new user’s home directory when `useradd -m` is used.

### Common Files:
- `.bashrc`
- `.bash_profile`
- `.profile`

### Behavior:
```bash
useradd -m alice
# /home/alice/ gets initialized with files from /etc/skel/
```


---


---

### 2. User Management Commands

| Command         | Purpose                                     |
|----------------|---------------------------------------------|
| `useradd`      | Create a user (low-level tool)              |
| `adduser`      | Interactive wrapper for `useradd` (Debian)  |
| `usermod`      | Modify existing user                        |
| `userdel`      | Delete user                                 |
| `passwd`       | Set or change password                      |
| `id`           | Display UID, GID, and groups of a user      |
| `whoami`       | Print effective current user                |
| `who`, `w`     | Show logged-in users                        |
| `last`         | Display login history                       |
| `chfn`, `chsh` | Change user info/shell                      |

---

### 3. Group Management

| Command        | Purpose                                    |
|----------------|--------------------------------------------|
| `groupadd`     | Add a new group                            |
| `groupdel`     | Delete a group                             |
| `groupmod`     | Modify group name or GID                   |
| `gpasswd`      | Set group admin/password                   |
| `groups`       | Display groups for a user                  |
| `newgrp`       | Temporarily switch to a different group    |
| `chgrp`        | Change group ownership of a file           |

---

### 4. File Ownership and Permissions

#### Permissions
- **User (u)**: File owner
- **Group (g)**: Group associated
- **Others (o)**: Everyone else

| Symbol | Meaning       |
|--------|---------------|
| `r`    | Read          |
| `w`    | Write         |
| `x`    | Execute       |
| `-`    | No permission |

#### Commands
- `chmod`: Change permission (numeric or symbolic)
- `chown`: Change ownership (user and optionally group)
- `chgrp`: Change group ownership

#### Special Permissions
- **SUID** (`chmod u+s file`): Executes with owner's privileges.
- **SGID** (`chmod g+s dir/file`):
  - On file: executes with group privileges.
  - On directory: new files inherit group.
- **Sticky bit** (`chmod +t dir`): Only owner can delete files inside a directory.

---

### 5. Home Directory Management

- `useradd -m`: Create home directory
- `useradd -d /path`: Set custom home dir
- `userdel -r`: Delete user and their home
- `/etc/skel/`: Default files copied to new home

---

### 6. Password and Account Policies

#### `passwd` Options
- `passwd user`: Change password
- `passwd -l user`: Lock account
- `passwd -u user`: Unlock
- `passwd -e user`: Force change at next login

#### `chage` Options
- `chage -l user`: View password aging
- `chage -M 90`: Max days between password changes
- `chage -m 7`: Min days before change allowed
- `chage -W 7`: Warning before password expiry
- `chage -I 30`: Inactive days after expiry

#### `/etc/login.defs`
- `PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `UID_MIN`, etc.

---

### 7. Login Restrictions and Session Control

#### PAM (Pluggable Authentication Modules)
- Configs in `/etc/pam.d/`
- Control authentication, password policy, session management

#### Common Modules
- `pam_unix.so`: Traditional UNIX auth
- `pam_limits.so`: Enforce limits.conf
- `pam_tally2.so`, `pam_faillock.so`: Failed login tracking

#### Files
- `/etc/nologin`: If present, denies non-root logins
- `/etc/security/limits.conf`: Set per-user limits (max processes, open files)

#### Example:
```
john    hard    nproc     100
alice   soft    nofile    2048
```

---

### 8. Monitoring and Auditing Users

| Command     | Purpose                                   |
|-------------|-------------------------------------------|
| `w`         | Show who is logged in and what they're doing |
| `who`       | Show logged-in users                      |
| `users`     | Show usernames of logged-in users         |
| `last`      | Display login history                     |
| `lastlog`   | Last login of each user                   |
| `id`        | Show UID, GID, groups                     |
| `finger`    | Show user info (requires `finger` package) |

---

### 9. User Environment Setup

| File              | Scope        | Purpose                             |
|-------------------|--------------|-------------------------------------|
| `~/.bashrc`       | User         | Shell customization on interactive login |
| `~/.profile`      | User         | Env vars and startup apps           |
| `~/.bash_profile` | User (RH)    | Login shell setup (Red Hat family)  |
| `/etc/profile`    | System-wide  | Global shell setup for login shells |

---

### 10. Advanced Topics

#### Access Control Lists (ACLs)
- For more granular permissions beyond rwx
- Commands:
  - `getfacl filename`
  - `setfacl -m u:alice:r file`

#### SELinux / AppArmor
- Mandatory Access Control systems
- Control how apps interact with files even if Unix permissions allow

#### User Namespaces
- Used in containers and sandboxes
- Map UID/GID inside isolated space to host users

---

