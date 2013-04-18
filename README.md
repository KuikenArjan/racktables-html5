RackTables - HTML5
================================

Thank you for selecting RackTables as your datacenter management solution!
If you are looking for documentation or wish to send feedback, please
look for the respective links at project's web-site (racktables.org).

Installing Racktables
---------------------

RackTables requires a MySQL server version 5.x built with InnoDB and Unicode support and configured appropriately. It also requires an Apache httpd with PHP 5 module and several PHP extensions. Below is a list of known-good distributions with respective setup notes.

### I. SERVERS

#### Fedora 8-16
* MySQL: yum install mysql-server mysql
* Apache/PHP: yum install httpd php php-mysql php-pdo php-gd php-snmp php-mbstring php-bcmath
* To enable Unicode, add "character-set-server=utf8" line to "[mysqld]" section of "/etc/my.cnf" file and restart mysqld.

#### Debian 6
* MySQL: aptitude install mysql-server-5.1
* Apache/PHP: aptitude install libapache2-mod-php5 php5-gd php5-mysql php5-snmp
* To enable Unicode, add "character-set-server=utf8" line to "[mysqld]" section of "/etc/mysql/my.cnf" file and restart mysqld.

#### ALTLinux 4.0
* MySQL: apt-get install MySQL-server
* Apache/PHP: apt-get install apache2-httpd-prefork php5-gd2 php5-pdo_mysql php5-pdo apache2-mod_php5 php5-mbstring
* To enable Unicode, add "CHSET=utf8" line to "/etc/sysconfig/mysqld" file and restart mysqld.

#### openSUSE 11.0
* MySQL: YaST -> Software -> software management -> Web and LAMP server -> mysql
* Apache/PHP: use YaST to install apache2-mod_php5, php5-gd, php5-mbstring, php5-mysql, php5-bcmath, php5-snmp and php5-ldap
* To enable Unicode, add "default-character-set=utf8" line to "[mysql]" section of "/etc/my.cnf" file and restart mysqld.

#### Scientific Linux 6
* MySQL: yum install mysql-server mysql
* Apache/PHP: httpd php php-mysql php-pdo php-gd php-mbstring php-bcmath
* To enable Unicode, add "character-set-server=utf8" line to "[mysqld]" section of "/etc/my.cnf" file and restart mysqld.

#### FreeBSD 8
* Apache/PHP:
 * make -C /usr/ports/www/apache13-modssl install
 * make -C /usr/ports/www/php5-session install
     - [X] CLI        Build CLI version
     - [X] APACHE     Build Apache module
     - [X] MULTIBYTE  Enable zend multibyte support
 * make -C /usr/ports/graphics/php5-gd install
 * make -C /usr/ports/databases/php5-pdo_mysql install
 * make -C /usr/ports/devel/pcre install
     - !!! Enable UTF-8 support ............ : yes
     - !!! Unicode properties .............. : yes
 * make -C /usr/ports/devel/php5-pcre install
 * make -C /usr/ports/converters/php5-mbstring install
      - [X] REGEX  Enable multibyte regex support
 * make -C /usr/ports/net-mgmt/php5-snmp install
 * make -C /usr/ports/net/php5-ldap install

### II. FILES
Unpack distro files to any directory you want and configure Apache to "wwwroot"
subdirectory as DocumentRoot.
Symlinks to wwwroot or even index.php from your web server root are also possible.

### III. INSTALLER
Open your configured RackTables URL and you will be prompted to configure
and initialize the application.

Upgrading Racktables
---------------------

RackTables (since 0.14.6) provides an automatic database upgrade feature.
If you already have a working installation, the following procedure
should be sufficient:

1. BACKUP YOUR DATABASE and check the release notes (below) before actually
   starting the upgrade,.
2. Remove all existing files except inc/secret.php and the gateways'
   configuration (in the gateways directory).
3. Unpack the new tarball into the place.
4. Open your RackTables page in a browser. The software detects version
   mismatch and displays a message telling to log in as admin to finish
   the upgrade.
5. Do that. Normally, everything should be Ok. If there are
   errors displayed, send these in a bug report.

Release Notes
---------------------

### Upgrading to 0.20.1

The 0.20.0 release includes bug which breaks IP networks' capacity displaying on 32-bit architecture machines. To fix this, this release makes use of PHP's BC Math module. It is a new reqiurement. Most PHP distributions have this module already enabled, but if yours does not - you need yo recompile PHP.

Security context of 'ipaddress' page now includes tags from the network containing an IP address. This means that you should audit your permission rules to check there is no unintended allows of changing IPs based on network's tagset. Example:
	allow {client network} and {New York}
This rule now not only allows any operation on NY client networks, but also any operation with IP addresses included in those networks. To fix this, you should change the rule this way:
	allow {client network} and {New York} and not {$page_ipaddress}

### Upgrading to 0.20.0

WARNING: This release have too many internal changes, some of them were waiting more than a year to be released. So this release is considered "BETA" and is recommended only to curiuos users, who agree to sacrifice the stability to the progress.

Racks and Rows are now stored in the database as Objects.  The RackObject table was renamed to Object.  SQL views were created to ease the migration of custom reports and scripts.

New plugins engine instead of local.php file. To make your own code stored in local.php work, you must move the local.php file into the plugins/ directory. The name of this file does not matter any more. You also can store multiple files in that dir, separate your plugins by features, share them and try the plugins from other people just placing them into plugins/ dir, no more merging. $path_to_local_php variable has no special meaning any more. $racktables_confdir variable is now used only to search for secret.php file. $racktables_plugins_dir is a new overridable special variable pointing to plugins/ directory. Beginning with this version it is possible to delete IP prefixes, VLANs, Virtual services and RS pools from within theirs properties tab. So please inspect your permissions rules to assure there are no undesired allows for deletion of these objects. To ensure this, you could try this code in the beginning of permissions script:

	allow {userid_1} and {$op_del}
	deny {$op_del} and ({$tab_edit} or {$tab_properties})

Hardware gateways engine was rewritten in this version of RackTables. This means that the file gateways/deviceconfig/switch.secrets.php is not used any more. To get information about configuring connection properties and credentials in a new way please visit http://wiki.racktables.org/index.php/Gateways

This also means that recently added features based on old API (D-Link switches and Linux gateway support contributed by Ilya Evseev) are not working any more and waiting to be forward-ported to new gateways API. Sorry for that.

Two new config variables appeared in this version:
  - SEARCH_DOMAINS. Comma-separated list of DNS domains which are considered "base" for your network. If RackTables search engine finds multiple objects based on your search input, but there is only one which FQDN consists of your input and one of these search domains, you will be redirected to this object and other results will be discarded. Such behavior was unconditional since 0.19.3, which caused many objections from users. So welcome this config var.
  - QUICK_LINK_PAGES. Comma-separated list of RackTables pages to display links to them on top. Each user could have his own list.

Also some of config variables have changed their default values in this version. This means that upgrade script will change their values if you have them in previous default state. This could be inconvenient, but it is the most effective way to encourage users to use new features. If this behavior is not what you want, simply revert these variables' values:

  - SHOW_LAST_TAB               no => yes
  - IPV4_TREE_SHOW_USAGE        yes =>no (networks' usage is still available by click)
  - IPV4LB_LISTSRC              {$typeid_4} => false
  - FILTER_DEFAULT_ANDOR        or => and (this implicitly enables the feature of dynamic tree shrinking)
  - FILTER_SUGGEST_EXTRA        no => yes (yes, we have extra logical filters!)
  - IPV4_TREE_RTR_AS_CELL       yes => no (display routers as simple text, not cell)

Also please note that variable IPV4_TREE_RTR_AS_CELL now has third special value besides 'yes' and 'no': 'none'. Use 'none' value if you are experiencing low performance on IP tree page. It will completely disable IP ranges scan for used/spare IPs and the speed of IP tree will increase radically. The price is you will not see the routers in IP tree at all.

### Upgrading to 0.19.13
A new "date" attribute type has been added. Existing date based fields ("HW warranty expiration", "support contract expiration" and "SW warranty expiration") will be converted to this new type but must be in the format "mm/dd/yyyy" otherwise the conversion will fail.

### Upgrading to 0.19.2

This release is different in filesystem layout. The "gateways" directory has been moved from "wwwroot" directory. This improves security a bit. You can also separate your local settings and add-ons from the core RackTables code. To do that, put a single index.php file into the DocumentRoot of your http server:

<?php
$racktables_confdir='/directory/where/your/secret.php/and/local.php/files/are/stored';
require '/directory_where_you_extracted_racktables_distro/wwwroot/index.php';
?>

No more files are needed to be available directly over the HTTP. 
Full list of filesystem paths which could be specified in custom index.php or secret.php:
 $racktables_gwdir:      path to the gateways directory;
 $racktables_staticdir:  path to the directory containing 'pix', 'js', 'css' dirs;
 $racktables_confdir:    path where secret.php and local.php are located. It is not recommended to define it in secret.php, cause only the path to local.php will be affected;
 $path_to_secret_php:    Ignore $racktables_confdir when locating secret.php and use the specified path;
 $path_to_local_php:     idem for local.php.

### Upgrading to 0.19.0

The files, which are intended for the httpd (web-server) directory, are now in the "wwwroot" directory of the tar.gz archive. Files outside of that directory are not directly intended for httpd environment and should not be copied to the server.

This release incorporates ObjectLog functionality, which used to be available as a separate plugin. For the best results it is advised to disable (through local.php) external ObjectLog plugin permanently before the new version is installed. All previously accumulated ObjectLog records will be available through the updated standard interface.

RackTables is now using PHP JSON extension which is included in the PHP core since 5.2.0.

The barcode attribute was removed. The upgrade script attempts to preserve the data by moving it to either the 'OEM S/N 1' attribute or to a Log entry. You should backup your database beforehand anyway.

### Upgrading to 0.18.x

RackTables from its version 0.18.0 and later is not compatible with RHEL/CentOS (at least with versions up to 5.5) Linux distributions in their default installation. There are yet options to work around that:
1. Install RackTables on a server with a different distribution/OS.
2. Request Linux distribution vendor to fix the bug with PCRE.
3. Repair your RHEL/CentOS installation yourself by fixing its PCRE RPM as explained here: http://bugs.centos.org/view.php?id=3252

### Upgrading to 0.17.0

One can always install RackTables 0.17.0 from scratch. However, upgrading an existing installation to 0.17.0 implies a certain upgrade path. If the existing database version is less, than 0.16.4, it must first be upgraded to version 0.16.4, 0.16.5 or 0.16.6 (at one's choice) using appropriate tar.gz distribution. The resulting 0.16.4+ database can be upgraded to 0.17.0 (or later version) in a normal way (with tar.gz of the desired 0.17.x release).

LDAP options have been moved to LDAP_options array. This means, that if you were using LDAP authentication for users in version 0.16.x, it will break right after upgrade to 0.17.0. To get things working again, adjust existing secret.php file according to secret-sample.php file provided with 0.17.0 release. This release is the first to take advantage of the foreign key support provided by the InnoDB storage engine in MySQL.  The installer and upgrader scripts check for InnoDB support and cannot complete without it. If you have trouble, the first step is to make sure the 'skip-innodb' option in my.cnf is commented out.

Another change is the addition of support for file uploads.  Files are stored in the database.  There are several settings in php.ini which you may need to modify:

    file_uploads        - needs to be On
    upload_max_filesize - max size for uploaded files
    post_max_size       - max size of all form data submitted via POST (including files)

User accounts used to have 'enabled' flag, which allowed individual blocking and unblocking of each. This flag was dropped in favor of existing mean of access setup (RackCode). An unconditional denying rule is automatically added into RackCode for such blocked account, so the effective security policy remains the same.
