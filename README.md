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

#### Essential Files
- `/etc/passwd`: Stores basic user account info. Format: `username:x:UID:GID:comment:home:shell`
- `/etc/shadow`: Stores encrypted passwords and aging info.
- `/etc/group`: Contains group name, GID, and members.
- `/etc/gshadow`: Secure version of `/etc/group` with password and admin fields.
- `/etc/login.defs`: Default settings for new user accounts.
- `/etc/skel/`: Template files copied to new user home directories.

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

