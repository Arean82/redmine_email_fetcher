Redmine Email Fetcher
=====================

This redmine plugin extends the Redmine [Receiving emails](http://www.redmine.org/projects/redmine/wiki/RedmineReceivingEmails#Fetching-emails-from-a-POP3-server) 
functionality by allowing the configuration of email accounts from where emails should be fetched periodically.

__Features__:

 * Stores IMAP and POP3 email configurations.
 * Associate each email configuration with the desired project, tracker and optionally with categories and priority.   
 * Allow manual email connection test and emails fetch.
 * A new task allows the cronjob to fetch emails from all active email configurations.
 * A configuration can be inactivate in order to stop its emails synchronization with Redmine.

__Remarks__:

* The plugin is prepared and intended to run with any IMAP and POP3 email account, however 
  some issues can accur due to security certificates.
* If you will be using SSL, please check that the machine has the proper certificates installed
  by running the following commands in the terminal:
  * `openssl`
  * `s_client -connect HOST:PORT`
* The Categories and Priority fields are free text, so if you update their name is your responsibility
  to update them accordingly in this plugin administration

Installation & Upgrade
----------------------

### Install

1. **install.** - Copy your plugin directory into `#{RAILS_ROOT}/plugins`.
   If you are downloading the plugin directly from GitHub, you can do so by
   changing into the `#{RAILS_ROOT}/plugins` directory and issuing the command:
   ```
   git clone git://github.com/luismaia/redmine_email_fetcher.git
   ```

### Upgrade

   **upgrade** - Backup and replace the old plugin directory with the new
   plugin files. If you are downloading the plugin directly from GitHub, you
   can do so by changing into the plugin directory and issuing the command
   `git pull`.

1. Update the ruby gems by changing into the redmine's directory and run the
   following command.
   ```
   bundle install
   ```

2. **upgrade** - Still on the redmine's directory, run the following command
   to upgrade your database (make a db backup before).
   ```
   rake redmine:plugins:migrate RAILS_ENV=production
   ```

3. Change into redmine's directory `#{RAILS_ROOT}` and run the following
   command.
   ```
   rake -T redmine:plugins:email_fetcher RAILS_ENV=production
   ```
   If the installation/upgrade was successful you should now see the list of
   [Rake Tasks](#rake-tasks).

4. Restart Redmine.

You should now be able to see **Redmine Email Fetcher** listed among the plugins in
`Administration -> Plugins`.
You should now be able to see **Redmine Email Fetcher** in the administration main area `Administration -> Fetch emails`.

### Uninstall

1. Change into redmine's directory `#{RAILS_ROOT}` and run the following
   command to downgrade the database (make a db backup before):
   ```
   rake redmine:plugins:migrate NAME=email_fetcher VERSION=0 RAILS_ENV=production
   ```

2. Remove the plugin from the plugins folder: `#{RAILS_ROOT}/plugins`
3. Restart Redmine.

Usage
-----

### Configuration

Open `Administration > Fetch emails` to access the plugin configuration:

**Email Configuration Attributes:**

+ **Configuration type** - Type of configuration of an email account:
  - **IMAP**
  - **POP3**
+ **Is configuration active?** - Specify if this email account is active and should
  be synchronized:
  - **True** - Task `fetch_all_emails` should synchronize this email account 
  - **False** - Task `fetch_all_emails` should ignore this email account
+ **Last successfully fetch at** - (read only) Date and time of the last synchronization of this 
  email account with Redmine.


**Email Attributes:**

+ **Host** - The IMAP or POP3 server host. Eg, `127.0.0.1`.
+ **Port** - The IMAP or POP3 server port. Eg., `993`.
+ **SSL** - Use SSL? (Default: `True`) [Options: `True`, `False`]
+ **Username** - The IMAP or POP3 email account username. Eg, `it@domain.com`.
+ **Password** - The IMAP or POP3 email account password.
+ **Folder name** - The email folder name from where the emails should be fetched:
  - **IMAP** - Any folder name is possible, but the test function will validate that 
    this folder is reachable after login). Eg, `REDMINE`.
  - **POP3** - Since this is an old protocol, only the `INBOX` folder is allowed
    (in fact you can't change this option)
+ **On success move to folder (IMAP only)** - (optional) This IMAP attribute allow us to 
  configure to where should we move emails that were successfully received instead of 
  deleting them Eg, `ARCHIVE`.
+ **On failure move to folder (IMAP only)** - (optional) This IMAP attribute allow us to
  configure to where should we move emails that were ignored Eg, `IGNORED`.
+ **Use APOP? (POP3 only)** - This POP3 attribute allow us to specify if APOP 
  authentication should be used (default: `false`).
+ **On failure delete email (POP3 only)** - This POP3 attribute allow us to specify if 
  messages that could not be processed successfully should be deleted from the server 
  (default: `false` - default behaviour is to leave them on the server).


**Unknown Sender Actions (in Redmine):**

+ **Method for unknown users** - How to handle emails from an unknown user where 
  ACTION can be one of the following values:
  - **accept** - the sender is considered as an anonymous user and the email is accepted (default).
  - **ignore** - The email is ignored.
  - **create** - A user account is created for the sender (username/password are sent back to the user) 
    and the email is accepted.
+ **Use no_account_notice** - Suppressing account generation notification (default `False`).
+ **Use no_permission_check** - Disable permission checking when receiving the email (default `True`).
+ **Default group for new reporters** - (optional) Automatically adding new users to some group(s).
  Eg, `group1,group2`.


**Default Issue creation (in Redmine) Attributes:**

+ **Tracker** - Target tracker (in case no tracker is specified in email content).
+ **Category name** - (optional) Name of the default category (in case no category is specified in email content).
+ **Priority name** - (optional) Name of the default priority (in case no priority is specified in email content).
+ **allow_override** - (optional) Allow email content to override attributes specified by previous options
  ATTRS is a comma separated list of attributes. Eg, `project,tracker,category,priority`.
+ **Project** - Target default project (in case no project is specified in email content).


For more information on these option visit Redmine official wiki [RedmineReceivingEmails](http://www.redmine.org/projects/redmine/wiki/RedmineReceivingEmails) page.


### Rake tasks

The following tasks are available:

    # rake -T redmine:plugins:email_fetcher
    rake redmine:plugins:email_fetcher:fetch_all_emails   # Fetch all active email accounts emails to Redmine
    
This tasks can be used to do periodic synchronization.
For example:

    # etch all active email accounts emails to Redmine @ every 5 minutes
    */5 * * * *   www-data /usr/bin/rake -f /opt/redmine/Rakefile --silent redmine:plugins:email_fetcher:fetch_all_emails RAILS_ENV=production 2>&- 1>&-

The tasks recognize three environment variables:
+ **DRY_RUN** - Performs a run without changing the database.
+ **ACTIVATE_USERS** - Activates users if they're active on LDAP.
+ **LOG_LEVEL** - Controls the rake task verbosity.
  The possible values are:
  - **silent**: Nothing is written to the output.
  - **error**: Only errors are written to the output.
  - **change**: Only writes errors and changes made to the user/group's base.
  - **debug**: Detailed information about the execution is visible to help
               identify errors. This is the default value.


License
-------
This plugin is released under the GPL v3 license. See LICENSE for more information.


Contributing
------------
Please fell free to contribute by adding more features or solving issues.
All PR are very welcome.
