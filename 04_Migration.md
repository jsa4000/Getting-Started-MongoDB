# MongoDB

## Execution

    docker run -it -e DB_URL=mongodb://mongoUri:27017 -v /tmp/mongo:/etc/mongo mongo bash

## Environments

- Development

        mongodb://sadmin:password@mongodb-shard-00-00.net:27017,mongodb-shard-00-01.net:27017,mongodb-shard-00-02.net:27017/test?ssl=true&replicaSet=mongodb-shard-0&authSource=admin

    i.e

        docker run -it -e DB_URL="mongodb://sadmin:password@mongodb-shard-00-00.net:27017,mongodb-shard-00-01.net:27017,mongodb-shard-00-02.net:27017/test?ssl=true&replicaSet=mongodb-shard-0&authSource=admin" -v /tmp/mongo:/etc/mongo mongo bash

- Staging

        mongodb://sadmin:password@homewifi-fon-hw-stg-shard-00-00.net:27017,homewifi-fon-hw-stg-shard-00-01.net:27017,homewifi-fon-hw-stg-shard-00-02.net:27017/test?ssl=true&replicaSet=homewifi-fon-hw-stg-shard-0&authSource=admin

## Connect

- Execute docker using the desired environment

        docker run -it -e DB_URL="mongodb://sadmin:password@mongodb-shard-00-00.net:27017,mongodb-shard-00-01.net:27017,mongodb-shard-00-02.net:27017/test?ssl=true&replicaSet=mongodb-shard-0&authSource=admin" -v /tmp/mongo:/etc/mongo mongo bash

- Simple connection to the database

        mongo $DB_URL

- Switch to a database and get all elements of a collection

        use db_devices
        db.getCollection("status").find()
        db.status.find()

- Exist from mongodb cli

        exit

## Export Collections

- Execute the docker container using the database `db_devices` to be exported as default in the mongodb url `...mongodb.net:27017/db_devices?ssl...`.

        docker run -it -e DB_URL="mongodb://sadmin:password@mongodb-shard-00-00.net:27017,mongodb-shard-00-01.net:27017,mongodb-shard-00-02.net:27017/db_devices?ssl=true&replicaSet=mongodb-shard-0&authSource=admin" -v /tmp/mongo:/etc/mongo mongo bash

- To export a collection into a `json` file

        mongoexport --uri $DB_URL --collection status --out /etc/mongo/status.json

## Import Collections

- Execute the docker container using the database `db_devices` to be exported as default in the mongodb url `...mongodb.net:27017/db_devices?ssl...`.

        docker run -it -e DB_URL="mongodb://sadmin:password@mongodb-shard-00-00.net:27017,mongodb-shard-00-01.net:27017,mongodb-shard-00-02.net:27017/db_devices?ssl=true&replicaSet=mongodb-shard-0&authSource=admin" -v /tmp/mongo:/etc/mongo mongo bash

- To export a collection into a `json` file

        mongoimport --uri $DB_URL --collection status --file /etc/mongo/status.json
        
        
 ## Backup/Restore MongoDB tools

[Back Up and Restore with MongoDB Tools](https://docs.mongodb.com/manual/tutorial/backup-and-restore-tools/)

This tutorial describes the process for creating backups and restoring data using the utilities provided with MongoDB. The `mongodump` and `mongorestore` utilities work with `BSON` data dumps, and are useful for creating backups of **small** deployments.

> For **resilient** and **non-disruptive** backups, use a file system or block-level disk snapshot function, such as the methods described in the MongoDB Backup Methods document.

`mongodump` only captures the documents in the database in its backup data and does not include **index** data. mongorestore or mongod must then rebuild the indexes after restoring data.

`mongorestore` and `--noIndexRestore` prevents mongorestore from restoring and building indexes as specified in the corresponding mongodump output

- Create dump for `db_devices`

        export DB_URL="mongodb://sadmin:password@mongodb-shard-00-00.net:27017,mongodb-shard-00-01.net:27017,mongodb-shard-00-02.net:27017/db_devices?ssl=true&replicaSet=mongodb-shard-0&authSource=admin"

        mongodump -v --uri $DB_URL --out=/tmp/mongodb/mongo-stg

        # Compress each dumped bson collection into a gzip file
        mongodump -v --uri $DB_URL --gzip --out=/tmp/mongodb/mongo-stg/

- Restore dump (`db_devices`)

  > mongorestore inferes automatically the path `/tmp/mongodb/mongo-stg/db_devices`

        export DB_URL="mongodb://sadmin:password@mongodb-shard-00-00.net:27017,mongodb-shard-00-01.net:27017,mongodb-shard-00-02.net:27017/db_devices?ssl=true&replicaSet=mongodb-shard-0&authSource=admin"

        mongorestore --uri $DB_URL -v --dryRun /tmp/mongodb/mongo-stg

- Test Restore (`--dryRun`)

        mongorestore --uri $DB_URL -v --dryRun --gzip --archive /etc/mongodb/db_collection_01.gz

## Backup/Restore with Filesystem Snapshots (wiredTiger)

[Back Up and Restore with Filesystem Snapshots](https://docs.mongodb.com/manual/tutorial/backup-with-filesystem-snapshots/)

These filesystem snapshots, or “block-level” backup methods, use system level tools to create copies of the device that holds MongoDB’s data files. These methods complete **quickly** and work **reliably**, but require additional system configuration outside of MongoDB.

*Changed in version 3.2*: MongoDB 3.2 added support for volume-level back up of MongoDB instances using the `WiredTiger` storage engine when the MongoDB instance’s data files and journal files reside on separate volumes. However, to create a coherent backup, the database must be locked and all writes to the database must be suspended during the backup process.

- Create a Snapshot¶

  To create a snapshot with `LVM`, issue a command as root in the following format:

                lvcreate --size 100M --snapshot --name mdb-snap01 /dev/vg0/mongodb

  > The snapshot has a cap of at 100 megabytes `--size 100M`

  This command creates an LVM snapshot (with the --snapshot option) named mdb-snap01 of the mongodb volume in the vg0 volume group. This example creates a snapshot named `mdb-snap01` located at `/dev/vg0/mdb-snap01`. The location and paths to your systems volume groups and devices may vary slightly depending on your operating system’s LVM configuration.

- Archive a Snapshot

  After creating a snapshot, mount the snapshot and copy the data to separate storage. Your system might try to compress the backup images as you move them offline. Alternatively, take a block level copy of the snapshot image, such as with the following procedure:

        umount /dev/vg0/mdb-snap01
        dd if=/dev/vg0/mdb-snap01 | gzip > mdb-snap01.gz

  The above command sequence does the following:

  - Ensures that the `/dev/vg0/mdb-snap01` device is **not** mounted. Never take a block level copy of a filesystem or filesystem snapshot that is mounted.
  - Performs a **block-level** copy of the entire snapshot image using the `dd` command and compresses the result in a `gzipped` file in the current working directory.

- Restore a Snapshot

  To restore a snapshot created with LVM, issue the following sequence of commands:

        lvcreate --size 1G --name mdb-new vg0
        gzip -d -c mdb-snap01.gz | dd of=/dev/vg0/mdb-new
        mount /dev/vg0/mdb-new /srv/mongodb

  > The restored snapshot will have a stale `mongod.lock` file. If you do **not** remove this file from the snapshot, and MongoDB may assume that the stale lock file indicates an unclean shutdown. If you’re running with `storage.journal.enabled` enabled, and you do not use `db.fsyncLock()`, you do not need to remove the `mongod.lock` file. If you use `db.fsyncLock()` you will need to remove the lock.

- Restore Directly from a Snapshot (without `gzip`)

  To restore a backup without writing to a compressed gz file, use the following sequence of commands:

        umount /dev/vg0/mdb-snap01
        lvcreate --size 1G --name mdb-new vg0
        dd if=/dev/vg0/mdb-snap01 of=/dev/vg0/mdb-new
        mount /dev/vg0/mdb-new /srv/mongodb

- Flush writes to disk and lock the database to prevent further writes.

        db.fsyncLock();

        # Perform the backup operation described in Create a Snapshot.

        db.fsyncUnlock();
