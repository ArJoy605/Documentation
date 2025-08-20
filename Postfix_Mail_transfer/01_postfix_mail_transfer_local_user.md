#

## Postfix Mail Transfer Local User Setup Guide

### Overview of the Setup Process

We’ll configure Postfix as the mail transfer agent (MTA) to handle local email delivery between users Ani and Roy on your Oracle Linux system. The setup ensures emails sent to `ani@joy.local` and `roy@joy.local` (or other variations like `ani@testserver.joy.local`) are delivered to their respective mailboxes (`/var/spool/mail/ani` and `/var/spool/mail/roy`). No external servers or internet access are required after package installation. The steps are:

1. Update the system and install Postfix.
2. Configure Postfix with the updated `mydestination` for local delivery.
3. Create users Ani and Roy.
4. Install `mailx` for sending and reading emails.
5. Test email exchange between Ani and Roy.
6. Verify and troubleshoot the setup.

### Detailed Steps with Explanations

#### Step 1: Update the System and Install Postfix

- **Command**:

  ```bash
  sudo dnf update -y
  sudo dnf install postfix -y
  ```

- **Explanation**:
  - `sudo dnf update -y`: Updates all system packages on Oracle Linux 8.10 to their latest versions, ensuring compatibility and security.
  - `sudo dnf install postfix -y`: Installs Postfix, the MTA that queues and delivers emails locally between Ani and Roy. The `-y` flag skips confirmation prompts.
  - Oracle Linux’s Postfix installation doesn’t prompt for a configuration wizard, so we’ll configure it manually next.
  - **Why this step?** Postfix is essential for email delivery, and updating the system prevents dependency or security issues. If Postfix is already installed, `dnf install` skips it, but the update is still recommended.

#### Step 2: Configure Postfix for Local Delivery

- **Command**:
  - Open the Postfix configuration file: `sudo vi /etc/postfix/main.cf` (or `sudo nano /etc/postfix/main.cf` if you prefer).
  - Ensure or update the following lines (replacing or modifying existing ones):

    ```bash
    myhostname = testserver.joy.local
    mydomain = joy.local
    myorigin = $mydomain
    inet_interfaces = loopback-only
    mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
    ```

  - Set the mailname: `echo "joy.local" | sudo tee /etc/mailname`.
  - Enable and start Postfix:

    ```bash
    sudo systemctl enable postfix
    sudo systemctl start postfix
    ```

  - Verify Postfix is running: `sudo systemctl status postfix` (look for "active (running)").
- **Explanation**:
  - **Configuration Details**:
    - `myhostname = testserver.joy.local`: Matches your system’s hostname from `hostnamectl`. This identifies the server in email headers and logs (e.g., `From: ani@testserver.joy.local`).
    - `mydomain = joy.local`: Defines the domain for email addresses (e.g., `ani@joy.local`). Extracted from your hostname, it’s the default domain for local users.
    - `myorigin = $mydomain`: Ensures outgoing emails use `@joy.local`. For example, if Ani sends to `roy`, Postfix interprets it as `roy@joy.local`.
    - `inet_interfaces = loopback-only`: Restricts Postfix to the local interface (127.0.0.1), ensuring no external connections for security in this local-only setup.
    - `mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain`: Specifies domains Postfix accepts email for:
      - `$myhostname` (`testserver.joy.local`): Accepts emails like `ani@testserver.joy.local`.
      - `localhost.$mydomain` (`localhost.joy.local`): Accepts emails like `ani@localhost.joy.local`.
      - `localhost`: Accepts emails like `ani@localhost`.
      - `$mydomain` (`joy.local`): Accepts emails like `ani@joy.local` (the addition you requested).
    - This updated `mydestination` ensures Postfix delivers emails addressed to `ani@joy.local` or `roy@joy.local` to the correct mailboxes, which is likely how users will address emails.
  - `/etc/mailname`: Sets `joy.local` as the default domain, reinforcing `mydomain` for consistent addressing.
  - `systemctl enable postfix`: Configures Postfix to start on boot.
  - `systemctl start postfix`: Applies the updated configuration.
  - `systemctl status postfix`: Confirms Postfix is running without errors.
  - **Why this step?** The updated `mydestination` ensures Postfix recognizes `joy.local` as a valid local domain, allowing emails to `ani@joy.local` and `roy@joy.local` to deliver correctly. The other settings keep the setup local-only, secure, and aligned with your hostname (`testserver.joy.local`).

#### Step 3: Create Users Ani and Roy

- **Command**:

  ```bash
  sudo adduser ani
  sudo adduser roy
  ```

- **Explanation**:
  - `sudo adduser ani`: Creates a system user `ani` with a home directory, prompting for a password and optional details (e.g., full name). Repeat for `roy`.
  - Mailboxes (`/var/spool/mail/ani` and `/var/spool/mail/roy`) are created automatically when each user receives their first email, storing emails in mbox format.
  - **Why this step?** Postfix delivers emails to local user accounts, so Ani and Roy must exist as system users with mailboxes for email delivery.

#### Step 4: Install a Mail Client and Test Sending Emails

- **Command**:
  - Install `mailx`:

    ```bash
    sudo dnf install mailx -y
    ```

  - Test sending from Ani to Roy:

    ```bash
    su - ani
    echo "Hi Roy, this is a test email from Ani." | mail -s "Test Email" roy
    exit
    ```

  - Read as Roy:

    ```bash
    su - roy
    mail
    ```

    (In `mail`, type `h` to list emails, select a number to read, and `q` to quit.)
  - Reply from Roy to Ani:

    ```bash
    echo "Hi Ani, got your message!" | mail -s "Reply from Roy" ani
    exit
    ```

- **Explanation**:
  - `sudo dnf install mailx -y`: Installs `mailx`, which provides the `mail` command for sending and reading emails.
  - `su - ani`: Switches to Ani’s user account to simulate sending as Ani.
  - `echo "message" | mail -s "Test Email" roy`: Sends an email to `roy`, which Postfix interprets as `roy@joy.local` (due to `myorigin = $mydomain`). The email is delivered to `/var/spool/mail/roy`.
  - `su - roy` and `mail`: As Roy, the `mail` command reads `/var/spool/mail/roy`. The email from Ani should appear with a header like `From: ani@joy.local`.
  - The reply from Roy to `ani` (i.e., `ani@joy.local`) delivers to `/var/spool/mail/ani`.
  - **Why this step?** Postfix handles delivery, but `mailx` provides the interface for Ani and Roy to compose, send, and read emails. This tests the email flow with the updated `mydestination`.

#### Step 5: Verify and Troubleshoot

- **Command**:
  - Check mail logs: `sudo tail -f /var/log/maillog`.
  - Check the mail queue: `sudo mailq`.
  - Verify mailboxes: `ls -l /var/spool/mail/` (should show `ani` and `roy` owned by each user).
  - If issues occur:
    - Confirm Postfix is running: `sudo systemctl status postfix`.
    - Verify users: `id ani` and `id roy`.
    - Check mailbox permissions: `ls -l /var/spool/mail/` (fix with `sudo chown ani /var/spool/mail/ani` if needed).
- **Explanation**:
  - `/var/log/maillog`: Logs show delivery details. Look for `to=<roy@joy.local>, status=sent` for successful deliveries or errors like `unknown user` (user doesn’t exist) or `relay access denied` (domain not in `mydestination`).
  - `mailq`: Shows undelivered emails. An empty queue indicates successful delivery.
  - Mailbox permissions: Files like `/var/spool/mail/ani` should be owned by `ani` with permissions `rw-------`.
  - **Why this step?** Verifies the setup works, especially with the updated `mydestination` allowing `joy.local`. Logs and `mailq` help diagnose issues like misconfigured domains or permissions.

### Expected Outcome

- Ani can send emails to `roy@joy.local`, and Roy to `ani@joy.local`. Emails can also be addressed to `ani@testserver.joy.local` or `ani@localhost.joy.local` due to the `mydestination` settings.
- Emails are stored in `/var/spool/mail/ani` and `/var/spool/mail/roy`, accessible via the `mail` command.
- The setup is local-only, requiring no internet after installation.

### Why the Updated `mydestination` Matters

- The original `mydestination = $myhostname, localhost.$mydomain, localhost` allowed emails to `ani@testserver.joy.local`, `ani@localhost.joy.local`, and `ani@localhost`.
- Adding `$mydomain` (i.e., `joy.local`) ensures Postfix accepts emails to `ani@joy.local`, which is likely the most intuitive address for Ani and Roy to use. Without `$mydomain`, emails to `ani@joy.local` might be rejected with a “relay access denied” error.

### Additional Notes

- **If Postfix was previously configured**: The updated `main.cf` includes all necessary changes. If you’ve already set it up, just modify the `mydestination` line and restart Postfix (`sudo systemctl restart postfix`).
- **Testing variations**: Try sending to `roy@joy.local`, `roy@testserver.joy.local`, or just `roy` to confirm all work. The `myorigin` and `mydestination` settings ensure flexibility.
- **Extending the setup**: For aliases (e.g., redirecting mail), edit `/etc/aliases` and run `sudo newaliases`. For IMAP or graphical clients, consider adding Dovecot, but that’s beyond this scope.
- **Troubleshooting**:
  - If emails fail, check `/var/log/maillog` for errors (e.g., `sudo tail -f /var/log/maillog`).
  - If `mail` fails, ensure `mailx` is installed and mailboxes exist (`ls /var/spool/mail/`).
  - If Postfix doesn’t start, check `sudo systemctl status postfix` for config errors.
