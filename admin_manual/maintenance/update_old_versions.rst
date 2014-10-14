General Infos
=============

Setup of ownCloud 4.5.13
------------------------

-  setup ownCloud 4.5.13 with sqlite
-  create mysql database

::

    CREATE DATABASE owncloud;
    CREATE USER 'ocadmin'@'localhost' IDENTIFIED BY 'password';
    GRANT ALL PRIVILEGES ON owncloud . * TO 'ocadmin'@'localhost';

-  adjust config.php to use mysql database

::

      'dbtype' => 'mysql',
      'dbname' => 'owncloud',
      'dbhost' => '127.0.0.1',
      'dbtableprefix' => 'oc_',
      'dbuser' => 'ocadmin',
      'dbpassword' => 'password',

-  integrate backup data: restore mysql backup

::

    mysql -uocadmin -ppassword owncloud < owncloud.sql

-  integrate backup data: copy the data backup to the data folder

::

    cp -r backup/data/* owncloud/data/

Backups before each update
--------------------------

::

    cp -r owncloud owncloud_backup
    mysqldump -uroot owncloud > owncloud_backup.sql

Update ownCloud version 4.5.13
==============================

Setup ownCloud 4.5.13 and integrate backup data

Update to 5.0.17
----------------

-  disable audit\_logging
-  backup

::

    cp -r owncloud owncloud_backup_4
    mysqldump -uocadmin -ppassword owncloud > owncloud_backup_4.sql

-  remove code, leave config and data folder only

::

    shopt -s extglob
    rm -rf owncloud/!(data|config)

-  update to 5.0.17

::

    VERSION="5.0.17"

    echo "Update to $VERSION"
    wget download.owncloud.com/download/$VERSION/owncloud_enterprise-$VERSION.tar.bz2
    tar xf owncloud_enterprise-$VERSION.tar.bz2

Open ownCloud frontend in the browser to start the update.

Update to 6.0.5
---------------

-  backup

::

    cp -r owncloud owncloud_backup_5
    mysqldump -uocadmin -ppassword owncloud > owncloud_backup_5.sql

-  remove code, leave config and data folder only

::

    shopt -s extglob
    rm -rf owncloud/!(data|config)

-  update to 6.0.5

::

    VERSION="6.0.5"
    echo "Update to $VERSION"
    wget download.owncloud.com/download/$VERSION/owncloud_enterprise-$VERSION.tar.bz2
    tar xf owncloud_enterprise-$VERSION.tar.bz2

Open ownCloud frontend in the browser to start the update.

Update ownCloud version 5.0.9
=============================

For testing the update setup a clone and update there

-  setup and install new 5.0.9
-  dump db backup
-  copy data folder from backup
-  check oc\_storages and fix manualy ( see notes on issue below )

Update to 5.0.17
----------------

-  remove code, leave config and data folder only
-  update to 5.0.17

Update to 6.0.5
---------------

-  remove code, leave config and data folder only
-  update to 6.0.5

Issue with absolute path entires in oc\_storages
------------------------------------------------

In versions 5.x a database table "oc\_storages" has absolute pathes to
the users "home" storage, like: ``local::/var/www/owncloud/data/admin/``

Normally the entries look like ``local::$datadir/$user/`` but if the
string is too long to fit the column md5 hashes are used instead

If you move the data folder, these entries point to the wrong (old)
folder path and here begins the trouble.

The Versions 4.0 and 4.5 have no oc\_storages table. So updates from 4.5
to 5.0.17 work fine, as the 5.0.17 does create relative pathes (is
fixed), like: ``home::admin``

In my test on updates, all works if I don't move the data folder at all.
Update from 4.0 to 4.5.13 to 5.0.17 work fine.

Older version 5.x (< 5.0.13) need to be migrated with care. - try to not
move the data folder - check the oc\_storages table - an update to
5.0.17 does not fix the absolute pathes - a following update to 6.0.5
does not fix the absolute pathes

So the entries need to be fixed manually. Please contact me if you have
tables with both, home:: and local:: entries.

This repair script can be used if only single ``local::...`` entries per user exist,
handle with care!

::

    #!/bin/sh

    for i in $(mysql -uroot owncloud -e 'select id from oc_storages where id like "local::/%";');
    do
      if [ $i == "id" ]; then continue; fi;
      echo $i
      
      # local::/var/www/owncloud/data/name/ --> home::name
      [[ $i =~ local::.*\/([^\/]+)\/ ]] && name="home::${BASH_REMATCH[1]}"
      
      echo $name
      mysql -uroot owncloud -e "update oc_storages set id = '$name' where id = '$i'"
    done

