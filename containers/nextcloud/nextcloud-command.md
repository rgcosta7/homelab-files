Nextcloud provides a command line tool (occ) that can be used to re-index user files. Any new files and directories added under a users account and not done so by the nextclould web interface e.g. from the command line, will be displayed in that users account.

The command to do this is:

~~~bash
sudo -u www-data php /var/www/nextcloud/occ files:scan --all
~~~