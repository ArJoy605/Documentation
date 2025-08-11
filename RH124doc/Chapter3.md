# RH124 Revision Notebook: Chapter 3 - Manage Files from the Command Line

This notebook provides a detailed summary of Chapter 3 from the Red Hat System Administration I (RH124) course, based on Red Hat Enterprise Linux (RHEL) 9/10. It focuses on managing files and directories via the command line, with practical commands, examples, common pitfalls, best practices, and revision exercises for effective learning and real-world application.

## Chapter 3: Manage Files from the Command Line

### Key Concepts

- **File System Hierarchy**: RHEL adheres to the Linux Filesystem Hierarchy Standard (FHS), organizing files in a predictable structure:
  - `/`: Root directory, the top of the hierarchy.
  - `/home`: User home directories (e.g., `/home/user1`).
  - `/etc`: Configuration files (e.g., `/etc/passwd` for user info).
  - `/var`: Variable data, such as logs (`/var/log/messages`) or temporary files (`/var/tmp`).
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
  - **Special Paths**: `.` (current directory), `..` (parent directory), `~` (user’s home directory).
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

- **Self-Test**:
  - Explain the output of `ls -l` (columns: permissions, links, owner, group, size, date, name).
  - Why use `mkdir -p`? (Creates parent directories if they don’t exist, avoids errors.)

---
