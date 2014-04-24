dovecot-mbc
===========

Dovecot plugin for executing a script on mailbox creation

Description
-----------
dovecot-mbc is a plugin for Dovecot

It can be used in order to let dovecot trigger a custom script, whenever a mailbox is created or renamed.
You could use this for instance in order to automatically set ACLs for public mailboxes or send a welcome message.

The script can be defined within the plugin's configuration and is provided with the following environment variables:
- MBC_MAILBOX		mailbox name
- MBC_DIRECTORY		mailbox directory
- MBC_TYPE		mailbox namespace type (private/shared/public)
- MBC_PREFIX		mailbox namespace prefix
- MBC_INBOX		mailbox namespace is inbox (true/false)
- MBC_HIDDEN		mailbox namespace hidden flag (true/false)
- MBC_SUBSCRIPTIONS	mailbox namespace handles subscriptions (true/false)
- MBC_LIST		mailbox namespace is listed (yes/no/children)

However, be reminded that the namespace information is depending on the context.
If for example the mailbox is created by the LDA, it may describe a namespace as "private", which is described as "public" when triggered via IMAP with a user context.
So please take this into consideration when your script is triggered in multiple contexts.

License
-------
GPLv3 (cheer for the FSF - Free Software means Free Society!)
For the notify-plugin.h file, it is LGPLv2.1

Installation
------------
Installation of the plugin is essentially a 4-step process:

1) Configure the paths in Makefile according to your installation
   The relevant variables are:

   * SCRIPT_GROUP - the system group of the main mail user/users
   * DOVECOT_INCDIR - directory containing dovecot header files
   * DOVECOT_MODULEDIR - directory where the plugin shall be installed
   * DOVECOT_ETCDIR - directory where dovecot.conf resides
   * DOVECOT_CONFIGDIR - dovecot's conf.d directory (or equivalent)
   * DOVECOT_PLUGIN_API_2_{0,1} - the plugin API to use
   * MAN1DIR - directory where the plugin's manual page shall be installed

2) Create a Makefile by issuing the following command

      ./configure

3) Compile the module with the following command line

      make build

3) Then copy the resulting binary mbc_plugin.so, the man page, the default config and example script to the corresponding directories
   This can be achieved using:

      make install

That's it.

Configuration
-------------
After the plugin library is installed and ready to be used, Dovecot's configuration needs to be adapted to make Dovecot use the plugin.

This is a 2-step process:

1) Make Dovecot aware of the plugin.
   This happens by adding it to the corresponding mail_plugins-list (e.g. global/lmtp/lda/imap/...)
   Note, that dovecot-mbc depends on the notify plugin's notifications, so don't forget to enable it as well!

      mail_plugins = $mail_plugins notify mbc

2) Setting the plugin's configuration options.
   This is done in the new configuration file conf.d/90-mbc.conf.

   Just provide the absolute path to the script you want to execute and you are set:

      plugin {
        mbc_script = /etc/dovecot/mbc_script.sh
      }

Now restart your dovecot daemon and the script is executed, if a context which uses the plugin creates or renames a mailbox.

GIT repository structure
---------------------
dovecot-mbc's git repository consists of the following folders and files:
- configure - the binary, which will create the makefile
- configure.ac - the autoconf file (for any further developments)
- Makefile.in - the template for the makefile, which is generated by ./configure
- README.md - the file you are currently looking at ;)
- conf/ - here you find the configuration files for the plugin
-- 90-mbc.conf - example configuration file
- contrib/ - here you find all the contributed files
-- mbc_script.sh - example
-- src/ - here is some more contributed source code
--- notify-plugin.h - the notify's plugin header file (which is needed during compilation)
- man/ - here you can find the man pages
--- dovecot-mbc.1.in - template for the man page (which is created during compilation)
- src/ - the plugin's source code
-- mbc-plugin.c - the main source code
-- mbc-plugin.h - the header file

Special thanks
---------------------
Special thanks go to Peter Marschall <peter@adpm.de>.
Reading his github repository https://github.com/marschap/fetchmail_wakeup helped me a lot. Some of this work may be inspired by his plugin.
