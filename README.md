# SYSTEM ADMINISTRATION AND DEVELOPMENT
### EMAIL SERVER ON A LINUX/UBUNTU SERVER WITH APACHE, DOVECOT AND POSTFIX
### GET AN OFFICIAL EMAILS FOR COMPANY EMPLOYEES USING ROUNDCUBE CLIENT SIDE FOR MAILS
![Q](https://github.com/user-attachments/assets/575b2d2d-7268-409e-800d-583edf0d552f)
---
## Introduction

**Hello, community!** Today, I will demonstrate how to configure an email server for your company, allowing you to have your own official email address rather than using services like Gmail, Yahoo, or Outlook. This setup will grant you full control over your email system, without limitations on the size, bandwidth, or number of users. 

We will be using open-source technologies such as:
- **Postfix** (for sending emails),
- **Dovecot** (for retrieving emails),
- **Apache** (as a web server),
- **MySQL** (for the database),
- **Roundcube** (as the webmail client),
- **Ubuntu** (as the operating system).

By the end of this tutorial, you will have a fully functional email server, which can handle both incoming and outgoing emails with no third-party limitations. You’ll also be able to manage emails via the web interface using Roundcube.

---

## Prerequisites

### Hardware Requirements
Before proceeding, ensure you have the following:
1. **Ubuntu Server**: This tutorial uses Ubuntu 20.04 LTS, but the steps will work with newer versions as well.
2. **Static IP Address**: For your email server to be reachable on the web, you need a public static IP address.
3. **Domain Name**: It’s highly recommended to have a domain name to make your email server professional (e.g., `yourcompany.com`).
4. **DNS Configuration**: You’ll need to configure DNS records for your domain. This includes an **MX Record** pointing to your mail server, and **A Record** for your domain’s IP address.
5. **Basic Linux Knowledge**: Familiarity with the command line, and installing and configuring packages.

---

## Step 1: Update Your Server and Install Required Packages

The first step in setting up your email server is to ensure your server is up-to-date and that you have the necessary packages installed.

1. **Update and Upgrade Your System**:

    Before installing any packages, it’s always a good idea to update the system:

    ```bash
    sudo apt update
    sudo apt upgrade
    ```

    This will ensure all your software packages are current and any security patches have been applied.

2. **Install Required Packages**:

    We need Apache (web server), MySQL (database), PHP (to run Roundcube), and other PHP extensions required by Roundcube. Run the following command to install everything:

    ```bash
    sudo apt install apache2 mysql-server php php-mysql php-imap php-mbstring php-xml php-zip php-json libapache2-mod-php
    ```

    ### Explanation:
    - **Apache2**: Our web server, used to host the Roundcube webmail interface.
    - **MySQL**: Used for storing email data, like users, emails, and Roundcube’s settings.
    - **PHP**: A scripting language required for Roundcube’s web interface to function.
    - **PHP Extensions**: These are additional features PHP needs to communicate with the database and handle email-specific functions (like IMAP for retrieving mail).

---

## Step 2: Install and Configure Postfix (Mail Transfer Agent)

**Postfix** is the software that will handle the sending and receiving of emails. It is a widely used **Mail Transfer Agent (MTA)**, known for its performance and ease of configuration.

1. **Install Postfix**:

    To install Postfix, run:

    ```bash
    sudo apt install postfix
    ```

2. **Configuring Postfix**:

    During the installation process, you’ll be prompted to configure Postfix:
    - Choose **"Internet Site"** when asked for the mail server type.
    - For the **system mail name**, use your public IP address (or domain name if you have one, e.g., `mail.yourcompany.com`).

### Postfix Configuration Details:

Once installed, you need to make some additional changes to the Postfix configuration file. This file is where you define how Postfix behaves when sending or receiving emails.

1. **Edit the Postfix configuration file**:

    ```bash
    sudo nano /etc/postfix/main.cf
    ```

2. **Modify the following lines**:

    ```bash
    myhostname = management
    mydomain = management.com
    myorigin = $mydomain
    inet_interfaces = all
    inet_protocols = ipv4
    myhostname = management.co.tz
    mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
    smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination


    ```

    ### Explanation:
    - **myhostname**: The name of your mail server.
    - **mydomain**: Your domain name (e.g., `management.com`).
    - **myorigin**: This specifies the domain name that will appear in the "From" address when your server sends emails.
    - **inet_interfaces**: Specifies which network interfaces Postfix listens to for incoming mail. `all` means it listens on all network interfaces.
    - **inet_protocols**: Set to `ipv4` to force Postfix to use IPv4 only.

3. **Restart Postfix** to apply the changes:

    ```bash
    sudo systemctl restart postfix
    ```

---

## Step 3: Install and Configure Dovecot (IMAP/POP3 Server)

**Dovecot** is the software that will allow users to retrieve their emails using protocols like IMAP or POP3. While Postfix handles sending emails, Dovecot handles receiving and managing them.

1. **Install Dovecot**:

    Install Dovecot and the required protocols:

    ```bash
    sudo apt install dovecot-core dovecot-imapd dovecot-pop3d
    ```

2. **Configure Dovecot to Use Maildir Format**:

    To store emails efficiently, we’ll configure Dovecot to use the **Maildir** format, which stores each email as a separate file (more efficient than the older mbox format).

    - Open the Dovecot configuration file:

        ```bash
        sudo nano /etc/dovecot/conf.d/10-mail.conf
        ```

    - Set the `mail_location` to:

        ```bash
        mail_location = maildir:~/Maildir
        ```

3. **Restart Dovecot**:

    ```bash
    sudo systemctl restart dovecot
    ```

---

## Step 4: Install and Configure Roundcube

Now that Postfix and Dovecot are handling email transport and retrieval, we need to install **Roundcube**, a web-based email client that will allow you to manage your emails through a user-friendly web interface.

1. **Install Roundcube**:

    Roundcube is included in the Ubuntu repositories. To install it, run:

    ```bash
    sudo apt install roundcube roundcube-core roundcube-mysql
    ```

2. During the installation, you’ll be asked to configure the Roundcube database. Select "Yes" and enter the **MySQL root password** when prompted.

---

## Step 5: Configure Roundcube to Work with Postfix and Dovecot

1. **Edit Roundcube's configuration file**:

    Open the Roundcube configuration file:

    ```bash
    sudo nano /etc/roundcube/config.inc.php
    ```
- Set the following configuration parameters**:

    ```php
    $config['default_host'] = 'localhost';  // IMAP Server
    $config['default_port'] = 143;          // IMAP Port

    $config['smtp_server'] = 'localhost';   // SMTP Server (Postfix)
    $config['smtp_port'] = 25;              // SMTP Port

    $config['smtp_port'] = 25;              // Default SMTP port
    $config['smtp_user'] = '%u';            // Use the full email as the SMTP username
    $config['smtp_pass'] = '%p';            // Use the password provided by the user
    $config['mail_domain'] = 'management.co.tz';  // Set the mail domain
    $config['smtp_auth_type'] = 'LOGIN';    // Enable SMTP authentication

    ```

### Explanation:
- **default_host**: This is the IMAP server that Roundcube will connect to in order to retrieve emails (Dovecot in this case).
- **default_port**: IMAP typically runs on port 143 (for unencrypted traffic) or 993 (for SSL).
- **smtp_server**: Specifies the SMTP server to be used for sending emails (Postfix).
- **smtp_port**: SMTP typically runs on port 25.

---

## Step 6: Create Email Accounts

You will need to create system users for each person who will have an email account. These users will have a corresponding email address on your domain (e.g., `user1@yourcompany.com`).

1. **Create system users**:

    ```bash
    sudo adduser user1
    sudo adduser user2
    ```

    Each of these commands will create a new user (with a home directory), which can be used for email accounts.

2. **Create Maildir directories for the users**:

    ```bash
    sudo mkdir -p /home/user1/Maildir
    sudo mkdir -p /home/user2/Maildir
    sudo chown -R user1:user1 /home/user1/Maildir
    sudo chown -R user2:user2 /home/user2/Maildir
    sudo chmod -R 700 /home/user1/Maildir
    sudo chmod -R 700 /home/user2/Maildir
    ```

---

## Step 7: Configure Apache to Serve Roundcube

For Roundcube to be accessible via a web browser, we need to make it available through Apache.

a. **Create a symlink to Roundcube in the web root**:

    ```bash
    sudo ln -s /usr/share/roundcube /var/www/html/roundcube
    ```

b. **Enable the Roundcube Apache configuration**:

    ```bash
    sudo a2enconf roundcube
    sudo systemctl reload apache2
    ```
c. **uncomment these lines,  firstly open this file**
```sh
sudo gedit /etc/dovecot/conf.d/10-master.conf
```
- then uncomment
```php  
service pop3-login {
  inet_listener pop3 {
    port = 110
  }
  inet_listener pop3s {
    port = 995
    ssl = yes
  }
}


service imap-login {
  inet_listener imap {
    port = 143
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }
```

d. **This will auto-complete addresses without a domain.**
```bash
sudo gedit /etc/roundcube/config.inc.php
```
- then add these lines
```php
$config['mail_domain'] = '10.0.2.15'; 
$config['validate_email'] = false;
```

e. **restart services**
```bash

sudo systemctl restart dovecot
sudo systemctl restart apache2
sudo systemctl restart postfix
```
3. **access Roundcube at `http://10.0.2.15/roundcube`**
- Log in as user1 and send an email to user2.
- Log in as user2 and send an email to user3.
- Log in as user3 and send an email to user4.
- Log in as user4 and send an email to user1.

---
## Conclusion

You now have a fully functional email server! You can access Roundcube at `http://10.0.2.15/roundcube` and log in using the email accounts you created. Postfix will handle outgoing emails, Dovecot will retrieve incoming emails, and Roundcube provides an easy-to-use web interface for managing your inbox.

For additional security, consider setting up SSL certificates using **Let’s Encrypt**, and configuring **SPF**, **DKIM**, and **DMARC** to protect against email spoofing.

Feel free to comment or reach out if you encounter any issues or have additional questions!

**My Contacts**

**WhatsApp**  
+255675839840  
+255656848274

**YouTube**  
[Visit my YouTube Channel](https://www.youtube.com/channel/UCjepDdFYKzVHFiOhsiVVffQ)

**Telegram**  
+255656848274  
+255738144353

**PlayStore**  
[Visit my PlayStore Developer Page](https://play.google.com/store/apps/dev?id=7334720987169992827&hl=en_US&pli=1)

**GitHub**  
[Visit my GitHub](https://github.com/shamiraty/)


