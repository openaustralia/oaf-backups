A simple script for dumping all the mysql and postgres databases on an Ubuntu box and backing them up to S3 with https://github.com/zertrin/duplicity-backup

Going to be using this for the OpenAustralia Foundation

Before you start:
cp duplicity-backup.conf.example duplicity-backup.conf

And set
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
PASSPHRASE
in duplicity-backup.conf

For the time being this script is on the server in the directory /home/matthewl/database-backup

To verify and see what's going on with the backups:

sudo ./duplicity-backup.sh -v    verifies the backup
sudo ./duplicity-backup.sh -l    lists the files currently backed up in the archive
sudo ./duplicity-backup.sh -s    show all the backup sets in the archive


In an EMERGENCY:

Restore a specific file
sudo ./duplicity-backup.sh --restore-file [FILE_TO_RESTORE] [DESTINATION]   

Restores the entire backup to [path]
sudo ./duplicity-backup.sh --restore [PATH]
