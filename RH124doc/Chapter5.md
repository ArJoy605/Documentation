# Chapter 5: Linux System Administration (RH124) - Detailed Notes

This chapter covers essential Linux administration concepts, focusing on input/output redirection, pipelines, text editing with Vim, and shell environment configuration. Below is a detailed breakdown of each topic.

---

## 1. Redirecting Output to a File or Program

Linux uses three standard streams for input and output:

- **Standard Input (stdin, file descriptor 0)**: The default source of input, typically the keyboard. Programs read user input or data from stdin.
- **Standard Output (stdout, file descriptor 1)**: The default destination for normal program output, typically displayed on the terminal.
- **Standard Error (stderr, file descriptor 2)**: The default destination for error messages, also typically displayed on the terminal.

### Redirection Operators

Redirection allows you to send these streams to files or other programs instead of their default destinations. The key operators are:

- **`>`**: Redirects stdout to a file, overwriting the file if it exists.
  - Example: `ls -l > output.txt` writes the directory listing to `output.txt`, replacing its contents.
- **`>>`**: Redirects stdout to a file, appending instead of overwriting.
  - Example: `ls -l >> output.txt` adds the directory listing to the end of `output.txt`.
- **`2>`**: Redirects stderr to a file, overwriting it.
  - Example: `ls /badpath 2> error.log` writes the error message to `error.log`.
- **`2>/dev/null`**: Discards stderr by redirecting it to `/dev/null`, a special file that discards all data.
  - Example: `ls /badpath 2>/dev/null` suppresses error messages.
- **`>file 2>&1`**: Redirects both stdout and stderr to the same file. The order is critical: stdout is redirected first, then stderr is redirected to the same destination as stdout.
  - Example: `ls -l /badpath > output.txt 2>&1` writes both output and errors to `output.txt`.
- **`&>file`**: A shorthand for redirecting both stdout and stderr to a file (equivalent to `>file 2>&1`).
  - Example: `ls -l /badpath &> output.txt`.
- **`&>>file`**: Appends both stdout and stderr to a file.
  - Example: `ls -l /badpath &>> output.txt`.

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

## 4. Editing Text Using Vim

**Vim** is a highly configurable, lightweight text editor commonly used in Linux environments. It’s powerful but has a learning curve due to its modal interface.

### Vim Operating Modes

Vim operates in several modes, each serving a specific purpose:

1. **Normal Mode** (default):
   - Used for navigation and issuing commands.
   - Examples: `h` (left), `j` (down), `k` (up), `l` (right), `dd` (delete line), `yy` (copy line).
   - Enter commands by typing `:` (e.g., `:w` to save, `:q` to quit).
2. **Insert Mode**:
   - Used for typing or editing text.
   - Enter from Normal Mode with keys like `i` (insert at cursor), `a` (append after cursor), `o` (new line below).
   - Exit to Normal Mode with `Esc`.
3. **Command-Line Mode**:
   - Used for executing complex commands, such as saving, quitting, or searching.
   - Enter by typing `:` in Normal Mode.
   - Examples: `:w` (save), `:q` (quit), `:wq` (save and quit), `:q!` (quit without saving).
4. **Visual Mode**:
   - Used for selecting text for operations like copying, cutting, or deleting.
   - Enter with:
     - `v`: Character-wise selection.
     - `V`: Line-wise selection.
     - `Ctrl+v`: Block-wise selection (useful for columnar editing).
   - Operations: `y` (yank/copy), `d` (delete), `p` (paste).

### Minimum and Basic Vim Workflow

1. **Open a file**: `vim filename`
2. **Edit**:
   - Enter Insert Mode: Press `i` to start typing.
   - Make changes to the text.
3. **Save and exit**:
   - Return to Normal Mode: Press `Esc`.
   - Save: Type `:w` and press `Enter`.
   - Quit: Type `:q` and press `Enter`.
   - Save and quit: Type `:wq` or `ZZ` (in Normal Mode).
4. **Quit without saving**:
   - Type `:q!` to discard changes and exit.

### Visual Mode in Vim

- **Purpose**: Select text for operations like copying, cutting, or formatting.
- **Usage**:
  - Enter Visual Mode: Press `v` (character-wise), `V` (line-wise), or `Ctrl+v` (block-wise).
  - Move the cursor to select text.
  - Perform an action:
    - `y`: Copy (yank) the selected text.
    - `d`: Delete (cut) the selected text.
    - `p`: Paste after the cursor.
  - Example:

    ```vim
    # In Normal Mode
    v           # Start character-wise selection
    <move>      # Move cursor to select text
    y           # Copy the selection
    p           # Paste the copied text
    ```

### Common Vim Commands

- Navigation: `h` (left), `j` (down), `k` (up), `l` (right), `w` (next word), `b` (previous word).
- Editing: `dd` (delete line), `yy` (copy line), `p` (paste), `u` (undo), `Ctrl+r` (redo).
- Search: `/pattern` (search forward), `?pattern` (search backward), `n` (next match), `N` (previous match).
- Replace: `:%s/old/new/g` (replace all occurrences of "old" with "new").

---

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
