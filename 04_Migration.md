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