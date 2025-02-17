# Postgres 16 Kubernetes Statefuleset

Known Issues, default postgres images will cause the permission issue like following error

***ERROR:***
```
chmod: changing permissions of '/var/lib/postgresql/data': Operation not permitted
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.utf8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

initdb: error: could not change permissions of directory "/var/lib/postgresql/data": Operation not permitted
fixing permissions on existing directory /var/lib/postgresql/data ... 
```

***Resolution***

Create custom docker image with the help of following command,push new created docker image to  your public/private docker registry and update docker image path in statefulet
```
docker build --no-cache -t docker.io/harishjadhav26/postgres:16 .

```

Update new docker image in postgres statefulset
```
.
.
   spec:
      containers:
      - name: postgres
        image: docker.io/harishjadhav26/postgres:16
.
.
```

