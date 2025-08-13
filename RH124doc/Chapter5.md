# Chapter 5: Linux System Administration (RH124) - Detailed Notes

This chapter covers essential Linux administration concepts, focusing on input/output redirection, pipelines, text editing with Vim, and shell environment configuration. Below is a detailed breakdown of each topic.

---

## 1. Redirecting Output to a File or Program

Linux uses three standard streams for input and output:

- **Standard Input (stdin, file descriptor 0)**: The default source of input, typically the keyboard. Programs read user input or data from stdin.
- **Standard Output (stdout, file descriptor 1)**: The default destination for normal program output, typically displayed on the terminal.
- **Standard Error (stderr, file descriptor 2)**: The default destination for error messages, also typically displayed on the terminal.

### Redirection Operators Table

The following table summarizes the key redirection operators used in Linux to manage standard output (stdout) and standard error (stderr) streams, redirecting them to files or other destinations.

| Operator          | Description                                                                 | Effect on File                     | Example                                      |
|-------------------|-----------------------------------------------------------------------------|------------------------------------|----------------------------------------------|
| `>`               | Redirects stdout to a file, overwriting the file if it exists.               | Overwrites file                    | `ls -l > output.txt`                         |
| `>>`              | Redirects stdout to a file, appending instead of overwriting.                | Appends to file                    | `ls -l >> output.txt`                        |
| `2>`              | Redirects stderr to a file, overwriting it.                                  | Overwrites file                    | `ls /badpath 2> error.log`                   |
| `2>/dev/null`     | Discards stderr by redirecting it to `/dev/null`.                           | Discards data                      | `ls /badpath 2>/dev/null`                    |
| `>file 2>&1`      | Redirects both stdout and stderr to the same file (order matters).           | Overwrites file                    | `ls -l /badpath > output.txt 2>&1`           |
| `&>file`          | Shorthand for redirecting both stdout and stderr to a file.                  | Overwrites file                    | `ls -l /badpath &> output.txt`               |
| `&>>file`         | Appends both stdout and stderr to a file.                                    | Appends to file                    | `ls -l /badpath &>> output.txt`              |

**Notes**:

- `/dev/null` is a special file that discards all data written to it, commonly used to suppress output.
- The `>file 2>&1` syntax ensures stdout is redirected first, then stderr is sent to the same destination as stdout.
- The `&>file` and `&>>file` are Bash-specific shorthands and may not work in all shells.

### Example of Output Redirection

```bash
# Redirect stdout to a file (overwrite)
ls -l /etc > etc_contents.txt

# Append stdout to a file
ls -l /var >> etc_contents.txt

# Redirect stderr to a file
ls /nonexistent 2> error.log

# Discard stderr
ls /nonexistent 2>/dev/null

# Redirect both stdout and stderr to a file
ls -l /nonexistent > output.txt 2>&1

# Shorthand for redirecting both stdout and stderr
ls -l /nonexistent &> output.txt
```

---

## 2. Constructing Pipelines

A **pipeline** connects the stdout of one command to the stdin of another, allowing data to flow between commands. This is achieved using the pipe operator (`|`).

- **Syntax**: `command1 | command2`
- **How it works**: The output of `command1` becomes the input for `command2`.
- **Use case**: Pipelines are ideal for processing data through multiple steps, such as filtering, sorting, or formatting.

### Examples of Pipelines

```bash
# List files and filter for those containing ".txt"
ls -l | grep ".txt"

# Read a file, sort its lines, and remove duplicates
cat names.txt | sort | uniq

# Count the number of files in a directory
ls /etc | wc -l
```

Pipelines can include multiple commands:

```bash
# List files, filter for ".conf", sort, and save to a file
ls /etc | grep ".conf" | sort > config_files.txt
```

---

## 3. Pipeline Using `tee`

The `tee` command reads from stdin and writes to both stdout and one or more files simultaneously. It’s useful for saving intermediate results in a pipeline while continuing to process the data.

- **Syntax**: `command1 | tee file | command2`
- **Options**:
  - `-a`: Append to the file instead of overwriting.
- **Use case**: Debugging pipelines or saving intermediate output for later analysis.

### Example of Using `tee`

```bash
# Save directory listing to a file and filter for ".log" files
ls -l /var/log | tee log_files.txt | grep ".log"

# Append to a file instead of overwriting
ls -l /etc | tee -a etc_files.txt | grep ".conf"
```

In the above example, `log_files.txt` contains the full output of `ls -l /var/log`, while the terminal shows only the lines containing `.log`.

---

## 4. Vim Cheatsheet

This cheatsheet provides a concise overview of Vim’s operating modes, basic workflow, and common commands for text editing in Linux.

### Vim Operating Modes

| Mode              | Purpose                                  | How to Enter                              | How to Exit            |
|-------------------|------------------------------------------|------------------------------------------|------------------------|
| **Normal Mode**   | Navigation and issuing commands          | Default mode, or `Esc` from other modes  | N/A                    |
| **Insert Mode**   | Typing or editing text                   | `i` (insert), `a` (append), `o` (new line below) | `Esc`                |
| **Command-Line Mode** | Executing commands (e.g., save, quit) | `:` in Normal Mode                       | `Enter` or `Esc`       |
| **Visual Mode**   | Selecting text for operations            | `v` (character-wise), `V` (line-wise), `Ctrl+v` (block-wise) | `Esc`            |

### Minimum Vim Workflow

| Step              | Action                                   | Command                                  |
|-------------------|------------------------------------------|------------------------------------------|
| Open a file       | Start editing a file                     | `vim filename`                           |
| Edit text         | Enter Insert Mode to type                | `i` (or `a`, `o`)                        |
| Save changes      | Write changes to file                    | `:w` (in Normal Mode)                    |
| Quit              | Exit Vim                                 | `:q` (in Normal Mode)                    |
| Save and quit     | Save changes and exit                    | `:wq` or `ZZ` (in Normal Mode)           |
| Quit without saving | Discard changes and exit                | `:q!` (in Normal Mode)                   |

### Visual Mode Operations

| Action            | Command                                  | Description                              |
|-------------------|------------------------------------------|------------------------------------------|
| Start selection   | `v`                                      | Character-wise selection                 |
|                   | `V`                                      | Line-wise selection                      |
|                   | `Ctrl+v`                                 | Block-wise selection (columnar editing)   |
| Copy (yank)       | `y`                                      | Copy selected text                       |
| Cut (delete)      | `d`                                      | Delete selected text                     |
| Paste             | `p`                                      | Paste after cursor                       |

**Example**: In Normal Mode, press `v`, move cursor to select text, press `y` to copy, then `p` to paste.

### Common Vim Commands

| Category          | Command                                  | Description                              |
|-------------------|------------------------------------------|------------------------------------------|
| **Navigation**    | `h`, `j`, `k`, `l`                       | Move left, down, up, right               |
|                   | `w`                                      | Move to next word                        |
|                   | `b`                                      | Move to previous word                    |
| **Editing**       | `dd`                                     | Delete current line                      |
|                   | `yy`                                     | Copy current line                        |
|                   | `p`                                      | Paste after cursor                       |
|                   | `u`                                      | Undo last change                         |
|                   | `Ctrl+r`                                 | Redo undone change                       |
| **Search**        | `/pattern`                               | Search forward for pattern               |
|                   | `?pattern`                               | Search backward for pattern              |
|                   | `n`                                      | Next match                               |
|                   | `N`                                      | Previous match                           |
| **Replace**       | `:%s/old/new/g`                          | Replace all occurrences of "old" with "new" |
| **Save/Quit**     | `:w`                                     | Save file                                |
|                   | `:q`                                     | Quit Vim                                 |
|                   | `:wq` or `ZZ`                            | Save and quit                            |
|                   | `:q!`                                    | Quit without saving                      |

**Notes**:

- Vim is modal, so always check your mode before typing.
- Use `Esc` to return to Normal Mode from Insert or Visual Mode.
- Save frequently with `:w` to avoid losing changes.
- Practice commands in a test file to build familiarity.

## 5. Changing the Shell Environment

The shell environment (typically Bash) can be customized using variables to control user sessions and program behavior.

### Assigning Values to Variables

- **Syntax**: `variable=value` (no spaces around `=`).
- **Rules**:
  - Variable names are case-sensitive and typically uppercase (e.g., `MYVAR`).
  - Values can be strings, numbers, or paths.
  - Enclose values in quotes if they contain spaces.
- **Example**:

  ```bash
  MYVAR="Hello, World!"
  PATH=/usr/local/bin:$PATH
  ```

### Retrieving Values with Variable Expansion

- Use `$variable` or `${variable}` to access a variable’s value.
- `${variable}` is preferred for clarity, especially when the variable name is adjacent to other characters.
- **Examples**:

  ```bash
  echo $MYVAR          # Outputs: Hello, World!
  echo ${MYVAR}        # Same as above
  echo ${MYVAR:-default}  # Outputs "default" if MYVAR is unset
  ```

### Configuring Bash with Shell Variables

- **Shell variables** customize the shell’s behavior and are local to the current session unless exported.
- Common shell variables:
  - `PS1`: Defines the shell prompt (e.g., `PS1='\u@\h:\w\$ '` for `username@hostname:directory$`).
  - `PATH`: Specifies directories to search for executable commands.
  - `HISTSIZE`: Sets the number of commands stored in the command history.
- **Example**:

  ```bash
  PS1='\u@\h:\w\$ '    # Customize the prompt
  HISTSIZE=2000        # Store 2000 commands in history
  ```

### Configuring Programs with Environment Variables

- **Environment variables** are shell variables exported to be available to child processes (e.g., programs launched from the shell).
- **Exporting**:
  - Use `export variable` to make a variable an environment variable.
  - Example: `export MYVAR` makes `MYVAR` available to subprocesses.
- Common environment variables:
  - `EDITOR`: Specifies the default text editor for programs like `crontab`.
  - `LANG`: Sets the language and locale (e.g., `LANG=en_US.UTF-8`).
- **Example**:

  ```bash
  export EDITOR=vim    # Sets Vim as the default editor
  export LANG=en_US.UTF-8
  ```

### Setting the Default Text Editor

- Set the `EDITOR` or `VISUAL` environment variable to specify the default editor.
- Persist this setting by adding it to `~/.bashrc` or `~/.bash_profile`.
- **Example**:

  ```bash
  export EDITOR=vim
  export VISUAL=vim
  ```

- Verify: `echo $EDITOR` should output `vim`.

### Unsetting and Unexporting Variables

- **Unsetting**: Removes a variable from the current session.
  - Syntax: `unset variable`
  - Example: `unset MYVAR` removes `MYVAR`.
- **Unexporting**: Makes an environment variable a shell variable (not available to child processes).
  - Syntax: `export -n variable`
  - Example: `export -n MYVAR` makes `MYVAR` a shell variable only.
- **Example**:

  ```bash
  MYVAR="Test"
  export MYVAR
  env | grep MYVAR     # Shows MYVAR in environment
  unset MYVAR          # Removes MYVAR
  export -n MYVAR      # Stops MYVAR from being an environment variable
  ```

### Persisting Variables

- To make variables persistent across sessions, add them to:
  - `~/.bashrc`: For user-specific shell variables.
  - `~/.bash_profile`: For user-specific environment variables (or login shell settings).
  - `/etc/environment` or `/etc/profile`: For system-wide environment variables.
- Example (add to `~/.bashrc`):

  ```bash
  export EDITOR=vim
  MYVAR="Persistent Value"
  ```

---
