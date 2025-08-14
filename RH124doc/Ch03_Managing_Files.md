# Chapter 3 - Manage Files from the Command Line

This notebook provides a detailed summary of Chapter 3 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on managing files and directories via the command line, with practical commands, examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 3: Manage Files from the Command Line

### Key Concepts

- **File System Hierarchy**: RHEL adheres to the Linux Filesystem Hierarchy Standard (FHS), organizing files in a predictable structure:
  - `/`: Root directory, the top of the hierarchy.
  - `/home`: User home directories (e.g., `/home/user1`).
  - `/etc`: Configuration files (e.g., `/etc/passwd` for user info).
  - `/var`: Variable data, such as logs (`/var/log/messages`) or temporary files (`/var/tmp`), printer spool files, mail spool files, databases, websites.
  - `/tmp`: Temporary files, typically cleared on reboot.
  - `/usr`: User-installed software, libraries, and documentation.
  - `/bin`, `/sbin`: Essential binaries for users (`/bin/ls`) and admins (`/sbin/fdisk`).
- **File Types**:
  - **Regular Files**: Text (e.g., `.txt`), binary (e.g., executables), or data (e.g., images, `.pdf`).
  - **Directories**: Containers for files and other directories.
  - **Symbolic Links**: Shortcuts pointing to another file or directory.
  - **Special Files**: Block devices (e.g., `/dev/sda` for disks), character devices, named pipes, or sockets.
  - Check file type with `ls -l` (first column: `-` for file, `d` for directory, `l` for link).
- **Path Navigation**:
  - **Absolute Paths**: Full path from root (e.g., `/var/log/messages`).
  - **Relative Paths**: Relative to current directory (e.g., `log/messages` if in `/var`).
  - **Special Paths**:
    - `.` (current directory)
    - `..` (parent directory)
    - `~` (user’s home directory).
- **File Operations**: Core tasks include creating, copying, moving, deleting, and linking files/directories, all performed via the command line for precision and automation.
- **Wildcards**: Pattern-matching for efficient file operations:
  - `*`: Matches any characters (e.g., `*.txt` for all `.txt` files).
  - `?`: Matches a single character (e.g., `file?.txt` matches `file1.txt` but not `file10.txt`).
  - `[abc]`: Matches one character from a set (e.g., `file[123].txt` matches `file1.txt`, `file2.txt`, `file3.txt`).
  - `[a-z]`: Matches a range (e.g., `file[a-c].txt` matches `filea.txt`, `fileb.txt`, `filec.txt`).
- **Case Sensitivity**: Linux file names are case-sensitive (e.g., `File.txt` ≠ `file.txt`).
- **Hidden Files**: Files starting with `.` (e.g., `.bashrc`) are hidden; view with `ls -a`.
- **RHEL 9/10 Notes**: Enhanced tools like `lsblk` for disk visibility and improved `find` performance for large file systems. RHEL 10 may integrate AI-assisted file management via Lightspeed (e.g., natural language queries for locating files).

### Important Commands

- **Listing Files**:

  ```bash
  ls  # List files in current directory
  ls -l  # Detailed list (permissions, owner, size, date)
  ls -a  # Include hidden files (e.g., .bashrc)
  ls -lh  # Human-readable sizes (e.g., 4K instead of 4096 bytes)
  ls -R  # Recursive list (includes subdirectories)
  ls *.txt  # List all .txt files using wildcard
  ls -alf  # All files, detailed, including hidden
  ```

- **Navigating Directories**:

  ```bash
  pwd  # Print Working Directory (e.g., /home/user)
  cd /var/log  # Change to absolute path
  cd logs  # Change to relative path (if in /var)
  cd ..  # Move to parent directory
  cd ~  # Go to home directory
  cd -  # Return to previous directory
  ```

- **Creating Files**:

  ```bash
  touch file.txt  # Create empty file or update timestamp
  touch file{1,2,3}.txt  # Create multiple files (file1.txt, file2.txt, file3.txt)
  echo "content" > file.txt  # Create/overwrite file with content
  echo "more content" >> file.txt  # Append content to file
  ```

- **Creating Directories**:

  ```bash
  mkdir mydir  # Create directory
  mkdir -p parent/child/grandchild  # Create nested directories
  ```

- **Copying Files/Directories**:

  ```bash
  cp file.txt file_copy.txt  # Copy file
  cp -r mydir mydir_copy  # Copy directory recursively
  cp *.txt /backup/  # Copy all .txt files to /backup
  cp -i file.txt /dest  # Prompt before overwriting
  ```

- **Moving/Renaming Files/Directories**:

  ```bash
  mv file.txt newfile.txt  # Rename file
  mv file.txt /new/location/  # Move file
  mv -i file.txt /dest  # Prompt before overwriting
  mv dir1 /new/location/dir2  # Move/rename directory
  ```

- **Deleting Files/Directories**:

  ```bash
  rm file.txt  # Delete file
  rm -r mydir  # Delete directory and contents
  rm -i *.txt  # Prompt before deleting each .txt file
  rm -f file.txt  # Force delete without prompt (use cautiously)
  rmdir ProjectY  # Remove empty directory
  ```

- **Creating Symbolic Links**:

  ```bash
  ln -s /path/to/file linkname  # Create symbolic link
  ln -s /var/log/messages messages_link  # Example link
  ```

- **Viewing File Types**:

  ```bash
  file file.txt  # Identify file type (e.g., ASCII text, binary)
  ls -l  # First column shows type (-: file, d: directory, l: link)
  ```

- **Finding Files** (Basic Introduction):

  ```bash
  find /home -name "file*.txt"  # Find files matching pattern
  find / -type d -name "log"  # Find directories named "log"
  ```

---

### Using `{}` to create multiple files

### Syntax

```bash
touch file{1,2,3}.txt
```

- This expands to: `file1.txt file2.txt file3.txt`
- It’s equivalent to running:

```bash
touch file1.txt file2.txt file3.txt
```

---

### More examples

1. **Number ranges:**

```bash
touch file{1..5}.txt
```

Creates:

```bash
file1.txt file2.txt file3.txt file4.txt file5.txt
```

1. **Letters:**

```bash
touch {a,b,c}.txt
```

Creates:

```bash
a.txt b.txt c.txt
```

1. **Combining letters and numbers:**

```bash
touch file{A,B,C}{1..3}.txt
```

Expands to:

```bash
fileA1.txt fileA2.txt fileA3.txt fileB1.txt fileB2.txt fileB3.txt fileC1.txt fileC2.txt fileC3.txt
```

1. **Creating directories:**

```bash
mkdir dir{1..3}
```

Creates: `dir1`, `dir2`, `dir3`

---

### Key points

- Works in **bash and most modern shells**.
- Saves you from typing long repetitive commands.
- Can be combined with **wildcards** and other shell expansions.

---

### Hard Links vs Soft Links (Symbolic Links)

| Aspect                               | Hard Link                                                                        | Soft Link (Symbolic Link)                                            |
| ------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| What it is                           | A directory entry pointing directly to the **inode** (data) of the original file | A special file that points to the **path/name** of the original file |
| Link to                              | The actual file data (inode)                                                     | The filename/path of the original file                               |
| Can link to                          | Only files on the **same filesystem**                                            | Files or directories across **different filesystems**                |
| Can link to directories?             | Usually **no** (except root or with special permissions)                         | Yes, you can link to directories                                     |
| Behavior if original file is deleted | The data **remains accessible** via the hard link                                | The link becomes **broken (dangling)**, points to nothing            |
| Inode number                         | Hard link shares the **same inode** as original file                             | Has a **different inode** (it's a separate file)                     |
| Usage                                | Makes multiple filenames for the **same file data**                              | Creates a shortcut/reference to the original file                    |

---

### How to create links

- **Hard link:**

  ```bash
  ln original.txt hardlink.txt
  ```

- **Soft link (symbolic link):**

  ```bash
  ln -s original.txt softlink.txt
  ```

---

### Example to illustrate

Suppose `file.txt` exists.

```bash
$ ls -li file.txt
12345 -rw-r--r-- 1 user user 100 Aug 13 09:00 file.txt

$ ln file.txt hardlink.txt
$ ln -s file.txt softlink.txt

$ ls -li file.txt hardlink.txt softlink.txt
12345 -rw-r--r-- 2 user user 100 Aug 13 09:00 file.txt
12345 -rw-r--r-- 2 user user 100 Aug 13 09:00 hardlink.txt
67890 lrwxrwxrwx 1 user user 8 Aug 13 09:05 softlink.txt -> file.txt
```

- `file.txt` and `hardlink.txt` share the **same inode (12345)**, so they are truly the same file data.
- `softlink.txt` has a different inode (`67890`) and points to `file.txt` by name.

---

### Key points of Hard Links vs Soft Links

- Deleting `file.txt`:

  - **Hard link:** You can still access the content via `hardlink.txt`.
  - **Soft link:** `softlink.txt` becomes broken, since it points to a non-existing file.

- Hard links cannot cross filesystem boundaries. Even if the original ﬁle gets deleted, the contents of the ﬁle are still available as long as at least one hard link exists. Data is only deleted from storage when the last hard link is deleted.
- Hard links have some limitations. Firstly, hard links can only be used with regular ﬁles. You cannot use `ln` to create a hard link to a directory or special ﬁle.
- Soft links can point to files or directories on different filesystems. If the original file is deleted, the soft link becomes a dangling link (points to nothing).
- Soft links can point anywhere, including directories or remote mounts.

---

Sure! Let’s go deeper into **globbing / pattern matching** in the shell, along with all the other expansions, so you get a more detailed view of how filenames can be matched and expanded.

---

### 1️⃣ **Globbing / Pattern Matching**

Globbing is the process where the shell **matches filenames** using wildcards. It happens **before the command is executed**.

### Basic Wildcards

| Wildcard | Matches                              | Example                                                    |
| -------- | ------------------------------------ | ---------------------------------------------------------- |
| `*`      | 0 or more characters                 | `ls *.txt` → all `.txt` files                              |
| `?`      | Exactly one character                | `ls file?.txt` → `file1.txt`, `fileA.txt`                  |
| `[abc]`  | Any one character in the set         | `ls file[12].txt` → `file1.txt` or `file2.txt`             |
| `[!abc]` | Any one character **not** in the set | `ls file[!1].txt` → all `fileX.txt` except `file1.txt`     |
| `[a-z]`  | Any one character in a range         | `ls file[a-c].txt` → `filea.txt`, `fileb.txt`, `filec.txt` |

---

### Advanced Globbing

1. **Character classes (`[:class:]`)**

```bash
ls file[[:digit:]].txt
# Matches file0.txt, file1.txt, ..., file9.txt
```

Common classes:

- `[:alnum:]` → letters & digits
- `[:alpha:]` → letters
- `[:digit:]` → digits
- `[:lower:]` → lowercase letters
- `[:upper:]` → uppercase letters
- `[:space:]` → space characters

1. **Extended Globbing** (enable with `shopt -s extglob`)

| Pattern      | Meaning                          |
| ------------ | -------------------------------- |
| `?(pattern)` | 0 or 1 occurrences of pattern    |
| `*(pattern)` | 0 or more occurrences of pattern |
| `+(pattern)` | 1 or more occurrences of pattern |
| `@(pattern)` | Exactly 1 occurrence of pattern  |
| `!(pattern)` | Anything except pattern          |

**Example:**

```bash
shopt -s extglob
ls +(*.txt|*.log)
# Lists all files ending with .txt or .log
```

---

### Notes on Globbing

- Only matches **existing filenames**; if no file matches, the pattern is returned as-is (can be changed with `nullglob`).
- Patterns are expanded **before variables, command substitution, or execution**.
- Can be used with `cp`, `mv`, `rm`, etc., to operate on multiple files easily.

---

### 2️⃣ **Tilde Expansion (`~`)**

- `~` → home directory of the current user
- `~username` → home directory of that user

```bash
cd ~          # /home/current_user
cd ~joy       # /home/joy
ls ~/Documents/*.txt
```

---

### 3️⃣ **Brace Expansion (`{}`)**

- Generates multiple strings from one expression.

```bash
touch file{A,B,C}{1..3}.txt
# fileA1.txt fileA2.txt fileA3.txt fileB1.txt ... fileC3.txt
```

- Can be combined with wildcards:

```bash
ls file{A,B}*.txt
# Matches all files starting with fileA or fileB and ending with .txt
```

---

### 4️⃣ **Variable Expansion (`$VAR`)**

```bash
F="file"
EXT="txt"
touch ${F}_1.${EXT} ${F}_2.${EXT}
# Creates file_1.txt and file_2.txt
```

- Supports **default values, string operations**, e.g., `${VAR:-default}`

---

### 5️⃣ **Command Substitution (`$(cmd)` or \`cmd\`)**

- Inserts the **output of a command** in place.

```bash
touch file_$(date +%Y%m%d).txt
# file_20250813.txt

LATEST=$(ls -t *.txt | head -1)
echo "Newest file: $LATEST"
```

---

### ✅ **Order of Shell Expansions**

When the shell processes a command:

1. **Tilde expansion** → `~` replaced by home directory
2. **Parameter / Variable expansion** → `$VAR`
3. **Command substitution** → `$(cmd)` or `` `cmd` ``
4. **Arithmetic expansion** → `$((expression))`
5. **Brace expansion** → `{a,b,c}`
6. **Globbing / Filename expansion** → `*`, `?`, `[abc]`

> So, globbing happens **after variables, commands, and braces** have been expanded.

---

### Example Combining Everything

```bash
USER="joy"
mkdir ~/projects/{test1,test2,test3}
touch ~/projects/{test1,test2,test3}/file_${USER}_$(date +%Y%m%d).txt
ls ~/projects/*/*.txt
```

- Expands braces → directories created
- Expands variables → `file_joy_...`
- Expands command substitution → date inserted
- Globbing lists all `.txt` files

---

If you want, I can make a **diagram showing all expansions step by step with examples**, which makes it very easy to memorize.

Do you want me to make that diagram?

### Practical Examples

- **Scenario: Organize Project Files**:
  - Create a project directory: `mkdir -p ~/projects/webapp`.
  - Create files: `touch ~/projects/webapp/{index.html,style.css,app.js}`.
  - Copy to backup: `cp -r ~/projects/webapp /backup/webapp_$(date +%F)`.
  - Verify: `ls -l /backup/webapp_2025-08-11`.
- **Scenario: Clean Up Logs**:
  - List old logs: `ls -l /var/log/*.log`.
  - Delete logs older than 30 days: `find /var/log -name "*.log" -mtime +30 -delete`.
  - Confirm deletion: `ls -l /var/log`.
- **Scenario: Link Configuration File**:
  - Create a link for easy access: `ln -s /etc/httpd/conf/httpd.conf ~/httpd.conf`.
  - Check link: `ls -l ~ | grep httpd.conf` (shows `l` for link).
- **Daily Use: Batch Rename**:
  - Rename all `.txt` to `.bak`: `for f in *.txt; do mv "$f" "${f%.txt}.bak"; done`.
- **Troubleshooting Example**: Accidentally deleted a file? Check if it’s still open by a process: `lsof | grep file.txt`, then recover from `/proc/<pid>/fd`.

### Common Pitfalls

- **Case Sensitivity**: `File.txt` ≠ `file.txt`. Always double-check names (use tab completion).
- **Overwriting Files**: `cp` or `mv` without `-i` can silently overwrite. Use `-i` for safety.
- **Recursive Deletion**: `rm -r` deletes directories and contents—double-check paths to avoid disasters (e.g., `rm -r /` is catastrophic).
- **Wildcard Errors**: `rm *.txt` in the wrong directory can delete unintended files. Verify with `ls *.txt` first.
- **Permission Issues**: Operations in `/etc` or `/var` often require `sudo` (e.g., `sudo cp file /etc/`).
- **Broken Links**: Symbolic links break if the target is moved/deleted. Verify with `ls -l` (broken links appear red in some terminals).

### Best Practices and Tips

- **Use Tab Completion**: Reduces typos in file names/paths (e.g., type `cd /v` and press Tab for `/var`).
- **Preview Actions**: Use `echo` with wildcards before destructive commands (e.g., `echo rm *.txt` to see matched files).
- **Backup Before Deletion**: Copy critical files to `/backup` or use `mv` instead of `rm` for recoverable deletes.
- **Organize Files**: Use meaningful directory structures (e.g., `/home/user/projects/webapp/{src,docs,tests}`).
- **Script Repetitive Tasks**: Save complex commands in scripts (e.g., `backup.sh` for copying logs).
- **Check Disk Space**: Before copying/moving large files, use `df -h` to ensure space.
- **RHEL 10 Tip**: Use Lightspeed for natural language file queries (e.g., `rhel lightspeed "find all log files in /var"`).

### Revision Quiz/Notes

- **Questions**:
  - What’s the difference between `cp -r` and `cp`? (`-r` copies directories recursively.)
  - How do you create three files at once? (`touch file{1..3}.txt`)
  - What happens if you run `rm -rf /tmp/*`? (Deletes all files in `/tmp` without prompting.)
- **Quick Exercise**:
  - Create a directory `~/test`, add two files (`data1.txt`, `data2.txt`), copy them to `~/test/backup`, and create a symbolic link to `data1.txt` named `link.txt`. Verify with `ls -l`.
  - Commands:

    ```bash
    mkdir -p ~/test/backup
    touch ~/test/{data1.txt,data2.txt}
    cp ~/test/data*.txt ~/test/backup
    ln -s ~/test/data1.txt ~/test/link.txt
    ls -l ~/test
    ```

---
