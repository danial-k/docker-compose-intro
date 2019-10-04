# Introduction
Wordpress is a blogging platform powered by a MySQL database.

# Starting the services
```shell
docker-compose up
```

Once all the services have started, create a new ```wordpress``` database from inside phpmyadmin by visiting http://127.0.0.1:9151 and logging in with ```root/root```.
Once the database has been created, the blog should be configurable at http://127.0.0.1:9150.
Once configured, the appropriate tables should be created by the Wordpress bootstrapper.
