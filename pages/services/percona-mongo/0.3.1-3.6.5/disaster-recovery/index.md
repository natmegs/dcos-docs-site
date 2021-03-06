---
layout: layout.pug
navigationTitle:  Disaster Recovery
title: Disaster Recovery
menuWeight: 80
excerpt:
featureMaturity:
enterprise: false
---

## Backup

The service supports a custom plan for creating a consistent backup of the MongoDB replica set that is uploaded to a remote location. The backup plan launches an instance of [Percona-Lab/mongodb_consistent_backup](https://github.com/Percona-Lab/mongodb_consistent_backup), creates a consistent backup of the replica set and uploads the backup to remote storage.

Currently only AWS S3 is supported as an upload destination for backups via the plan named 'backup-s3'. More upload methods coming in the future!

There are two ways to configure backups, via the DC/OS Percona-Mongo service configuration section 'Backup restore' in the DC/OS GUI and/or the DC/OS CLI.

### Hidden Secondary Member
The service supports the ability to launch a dedicated MongoDB [hidden secondary](https://docs.mongodb.com/manual/core/replica-set-hidden-member/) replica set member for purpose of performing backups.

[Hidden secondary](https://docs.mongodb.com/manual/core/replica-set-hidden-member/) members cannot become Primary in a failover situation and are hidden to application drivers.

The use of a dedicated backup secondary gurrantees backups *(a very resource-intensive process)* do not impact database nodes used by the application, therefore we recommend this is enabled when using backups. Enabling a hidden secondary will add a task named *'backup-0-mongod'* to the service. This node will inherit all settings from the other replica set nodes.

### Starting a Backup

Backups are started using the Percona-Mongo CLI module's `backup` command.

If the AWS Access Key, Secret Key, S3 Bucket Name and S3 Bucket Prefix are already defined in your service options the following will start a backup:
    ```shell
    $ dcos percona-mongo backup run s3
    ```

If the AWS configuration/credentials are NOT defined in the service options or you would like to override them, define the options on the command line with the following:
    ```shell
    $ dcos percona-mongo backup run s3 \
        --access-key=XXXXXXXXXXXXXXXXXXX \
        --secret-key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
        --bucket-name=my-s3-bucket-name \
        --bucket-prefix=/mongobackups \
        --region=eu-central-1
    ```

A successful start of the backup will return the following:

    ```javascript
    {
      "message": "Received cmd: start"
    }
    ```

A task named *'mongodb-consistent-backup-0-backup'* will exist for the duration of the backup and upload. In this example, a backup will be uploaded to *'s3://my-s3-bucket-name/mongobackups/<DATE>'* containing several subdirectories.

### Stopping a Backup

To stop a running backup process:

    ```shell
    $ dcos percona-mongo backup stop s3
    ```

### Troubleshooting a Backup

To troubleshoot problems with backups, add the flag *'--backup-verbose'* to the backup command. Please report any issues with mongodb_consistent_backup to [https://github.com/Percona-Lab/mongodb_consistent_backup/issues/new](https://github.com/Percona-Lab/mongodb_consistent_backup/issues/new).

## Restore

Running a restore of a mongodump-based backup stored on Amazon S3 is possible via the DC/OS GUI and CLI tool, including backups created by the service.

### Using Backups Created by Percona-Mongo

To restore an AWS S3-based backup that was created by the Percona-Mongo backup feature, note that the AWS S3 URL must point to the *'dump'* directory for the desired replica set.

As an example, to restore the replica set *'rs'* to a backup located at AWS S3 URL *'s3://my-s3-bucket-name-here/backup/20170618_1600'* the following URL should be provided to the Percona-Mongo Restore features: *'s3://my-s3-bucket-name-here/backup/20170618_1600/rs/dump'*.

### Restore at Replica Set Initiation

The service supports running a restore after the initiation of the MongoDB Replica Set. This is useful for migrations to the percona-mongo service, cloning environments, etc.

Steps:
1. In the DC/OS GUI, go to *'Catalog'*.
1. Find the *'percona-mongo'* service.
1. Press *'Review & Run'*.
1. Go to the *'Backup Restore'* tab in the service configuration. 
1. Set your AWS Access Key *(accessKey)* and Secret Key *(secretKey)* in the *'Aws s3 configuration'* section. This user must have access to the read the backup.
1. Enter the AWS S3 URL to the *'dump'* backup directory that was outputted by mongodump into the *'s3Url'* field.
1. Check the *'restoreAfterInit'* field, to enable the restore once the replica set has been initiated.
1. Wait for the task *'restore-0-restore-s3'* to reach the *'FINISHED'* state.

### Restore using the DC/OS CLI

A manual restore is started by using the Percona-Mongo CLI module's `restore` command.

If the AWS Access Key and Secret Keyare already defined in your service options, the following will start a restore:

    ```shell
    $ dcos percona-mongo restore run s3 s3://my-s3-bucket-name-here/backup/dump
    ```

If the AWS configuration/credentials are NOT defined in the service options or you would like to override them, define the options on the command line with the following:

    ```shell
    $ dcos percona-mongo restore run s3 \
        --access-key=XXXXXXXXXXXXXXXXXXX \
        --secret-key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
        s3://my-s3-bucket-name-here/backup/dump
    ```

### Stopping a Restore

To stop a running backup restore process:

    ```shell
    $ dcos percona-mongo restore stop s3
    ```

### Troubleshooting a Restore 

See the *'Logs'* page of the *'restore-0-restore-s3'* task to troubleshoot the restore process.
