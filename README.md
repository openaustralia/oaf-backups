> [!WARNING]
> This project has been archived because we are not actively maintaining nor using it. It hasn't seen activity since October 2017 and may have known security vulnerabilities that have not been addressed. If you would like to move this work forward, please do! If we get enough interest, we might reconsider working on this again. You can always donate to provide us with more resources.

Note: I (@benrfairless) used AI to assist with the process of archiving this repository, however I reviewed the decision myself before taking action.
# OpenAustralia Foundation Backups

This repository contains details of all of OpenAustralia Foundation's backups and how to restore them. It also contains a script we use on some servers to do database backups and store them on Amazon S3.

## Backup and restore procedures

These are listed by server as that's probably what you're interested in when you're looking at this document.

### kedumba

[`database-backup.sh`](#database-backupsh) is cloned at `/home/matthewl/database-backup`. See below for restore procedures.

### jamison

[`database-backup.sh`](#database-backupsh) is cloned at `/home/henare/database-backup`. See below for restore procedures.

### morph.io

#### Backups

MySQL is backed up locally using Xtrabackup.
This is [provisioned using an Ansible role](https://github.com/openaustralia/morph/tree/master/provisioning/roles/backups).
Xtrabackup [stores backups](https://github.com/openaustralia/morph/blob/master/provisioning/roles/backups/files/database-backup.sh) in `/backups/mysql/` and keeps a copy of the last 5 days of backups.

The SQLite scraper databases and the local Xtrabackup files are backed up to S3 using [Duply](http://www.duply.net/wiki/index.php/Duply-documentation), which is a nice frontend to Duplicity. This is [configured using Ansible](https://github.com/openaustralia/morph/blob/master/provisioning/roles/morph-app/tasks/main.yml#L168-L186).

#### Restoring

There are 2 duply profiles, `mysql` and `sqlite`. You can restore from S3 with them as follows:

```
# Restore a specific [file] to [destination] from [profile] on [date]
duply [profile] fetch [file] [destination] [date]

# Restore the entire latest backup from [profile] to [destination]
duply [profile] restore [destination]
```

TODO: Add documentation on how to restore Xtrabackup MySQL backups.

### cuttlefish.oaf.org.au

[`database-backup.sh`](#database-backupsh) is cloned at `/root/database-backup`. See below for restore procedures.

[`database-backup.sh` is provisioned](https://github.com/openaustralia/cuttlefish/blob/master/provisioning/roles/backup/tasks/main.yml) using Ansible.

Linode backups are also enabled for this server.

## database-backup.sh

This script is a little misleadingly named as it also backs up directories to S3. It is a simple script for dumping all the MySQL and PostgreSQL databases on an Ubuntu box and backing them up, and any directories you choose, to S3 with https://github.com/zertrin/duplicity-backup

### Setup

1. Clone the repository onto the server. The location of this is a bit haphazard at the moment but the most common location is someone's home directory. It would probably be best to standardise things and [put it in a dedicated root directory](https://github.com/openaustralia/oaf-backups/issues/4). `git clone https://github.com/openaustralia/oaf-backups.git`.

2. Copy the example config so you can configure things:
`cp duplicity-backup.conf.example duplicity-backup.conf`.

3. Now you need to configure a bunch of settings in `duplicity-backup.conf`. The first are the AWS credentials `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. You'll either need to generate a new key (beyond the scope of this document) or use the one we have already set up for `oaf-backups`.

4. Set the GPG passphrase, `PASSPHRASE`. This is used to encrypt the backup. If you lose it then there is no way to access the backup ever again.

5. Set backup destination, `DEST` to `s3+http://oaf-backups/[name_of_server]/`.

6. Set included list of directories, `INCLIST` to the location of the database backups directory (the default is currently `/var/lib/database-backup`) and any other directories you want backed up.

7. Now that your backups are configured you can test them by running `./database_backup.sh`.

8. If that succeeds then set that script to be run as a daily cronjob.

### Verifying and checking the backups

```
./duplicity-backup.sh -v    # verifies the backup
./duplicity-backup.sh -l    # lists the files currently backed up in the archive
./duplicity-backup.sh -s    # show all the backup sets in the archive
```

### Restoring files from backup

```
# Restore a specific file
./duplicity-backup.sh --restore-file [FILE_TO_RESTORE] [DESTINATION]

# Restores the entire backup to [path]
./duplicity-backup.sh --restore [PATH]
```

#### Restoring databases

MySQL databases are backed up using `mysqldump`. PostgreSQL databases are backed up using `pg_dump`. These create dump files that can be restored to their respective database.

The process for doing so isn't documented here because there's a variety of ways you might want to do this, depending on the failure you're recovering from.

## Copyright & License

Copyright OpenAustralia Foundation Limited. Licensed under the Affero GPL. See LICENSE file for more details.