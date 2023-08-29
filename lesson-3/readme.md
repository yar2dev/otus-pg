Установка PostgreSQL 

## вывод терминала



```bash
root@test-pg:/home/adm5# mkdir /var/lib/postgres
root@test-pg:/home/adm5# docker network create pg-local
3dad73f2c5a9726e2b481a3d2a37e1bf04d69f0ed472a8d0391b12f920a5e0d9
root@test-pg:/home/adm5# docker run --name pg-server --network pg-local -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
Unable to find image 'postgres:15' locally
15: Pulling from library/postgres
52d2b7f179e3: Pull complete 
d9c06b35c8a5: Pull complete 
ec0d4c36c7f4: Pull complete 
aa8e32a16a69: Pull complete 
8950a67e90d4: Pull complete 
1b47429b7c5f: Pull complete 
a773f7da97bb: Pull complete 
7bddc9bbcf13: Pull complete 
60829730fa39: Pull complete 
f3d9c845d2f3: Pull complete 
cfcd43fe346d: Pull complete 
576335d55cdb: Pull complete 
caad4144446c: Pull complete 
Digest: sha256:a5e89e5f2679863bedef929c4a7ec5d1a2cb3c045f13b47680d86f8701144ed7
Status: Downloaded newer image for postgres:15
7621ac4dff56dc4ef926fce9a3c0122051bb7cff6c5420de9a0cb4050e15ebce
root@test-pg:/home/adm5# docker run -it --rm --network pg-local --name pg-client postgres:15 psql -h pg-server -U postgres
Password for user postgres: 
psql (15.4 (Debian 15.4-1.pgdg120+1))
Type "help" for help.

postgres=# \l
                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
(3 rows)

postgres=# create database test;
CREATE DATABASE
postgres=# \l
                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 test      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
(4 rows)

postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# create table one (int i)
test-# insert insert into one(i) values('10');
ERROR:  syntax error at or near "insert"
LINE 2: insert insert into one(i) values('10');
        ^
test=# insert into one(i) values('10');
ERROR:  relation "one" does not exist
LINE 1: insert into one(i) values('10');
                    ^
test=# select * from one;
ERROR:  relation "one" does not exist
LINE 1: select * from one;
                      ^
test=# \df
                       List of functions
 Schema | Name | Result data type | Argument data types | Type 
--------+------+------------------+---------------------+------
(0 rows)

test=# create table one (int i);
ERROR:  type "i" does not exist
LINE 1: create table one (int i);
                              ^
test=# create table one (i int);
CREATE TABLE
test=# insert into one(i) values('10');
INSERT 0 1
test=# insert into one(i) values('20');
INSERT 0 1
test=# select * from one;
 i  
----
 10
 20
(2 rows)

test=# \q
root@test-pg:/home/adm5# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
7621ac4dff56   postgres:15   "docker-entrypoint.s…"   10 minutes ago   Up 10 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
root@test-pg:/home/adm5# docker stop pg-server
pg-server
root@test-pg:/home/adm5# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
postgres     15        43677b39c446   13 days ago   412MB
alpine       latest    7e01a0d0a1dc   3 weeks ago   7.34MB
root@test-pg:/home/adm5# docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Are you sure you want to continue? [y/N] y
Deleted Containers:
7621ac4dff56dc4ef926fce9a3c0122051bb7cff6c5420de9a0cb4050e15ebce
6751db825e8684d86adeac04894b73ed1f9bc2d3584312e1df9e9c6d96b36faa

Deleted Networks:
pg-local

Total reclaimed space: 0B
root@test-pg:/home/adm5# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
postgres     15        43677b39c446   13 days ago   412MB
alpine       latest    7e01a0d0a1dc   3 weeks ago   7.34MB
root@test-pg:/home/adm5# docker rmi -f $(docker images -aq)
Untagged: postgres:15
Untagged: postgres@sha256:a5e89e5f2679863bedef929c4a7ec5d1a2cb3c045f13b47680d86f8701144ed7
Deleted: sha256:43677b39c446545c6ce30dd8e32d8c0c0acd7c00eac768a834e10018f5e38c32
Deleted: sha256:67cb76f8afc496c6c399ed442a80becf8e1ed84725d9a002140a15043c8de06f
Deleted: sha256:fc3a102562d98a65585a3b5b453f36b6cebc509b900cb1581a3cf7b2a8792d57
Deleted: sha256:be87653794c7adfd62484c50a7c101b37f5719dfaa94c8f63b26c29e4e1bb9c3
Deleted: sha256:59e52d5b26e0260d51ff31a37fd09d4403e32387db1111061e9abfaaac0c3225
Deleted: sha256:e81d2074621167317dd796197226bc347c4cff582251f63ef6661eba1f333fca
Deleted: sha256:54d1188d0381d86e216039ef511c8320967e305fcf28542f1a8f6fe9f9845e50
Deleted: sha256:214fdccb284b4983b060a9b985233226fa98cc42cd128f7743bebd1304d05146
Deleted: sha256:8671b99aa7397db41312821820aba285f54320de2f8111d8dc77e5b0cd57416b
Deleted: sha256:143edf11cd60b76f3cb8a6f0bb21b9eb9f2be2cfd08cb452ca395274f54336a4
Deleted: sha256:9e98afa6d365837777a097d4185740d6fe5eb5d2294a675e03e57cb4cff870dc
Deleted: sha256:3a6626c0394ee8fb5c4c3592a9eb260c065cac49a0d8dbc3f69d487716aba3cc
Deleted: sha256:827cd90e6de8d299ddd3eeb49c82435e7b00d0c7f9c586e80dd2a5ea8409b0d6
Deleted: sha256:511780f88f80081112aea1bfdca6c800e1983e401b338e20b2c6e97f384e4299
Untagged: alpine:latest
Untagged: alpine@sha256:7144f7bab3d4c2648d7e59409f15ec52a18006a128c733fcff20d3a4a54ba44a
Deleted: sha256:7e01a0d0a1dcd9e539f8e9bbd80106d59efbdf97293b3d38f5d7a34501526cdb
Deleted: sha256:4693057ce2364720d39e57e85a5b8e0bd9ac3573716237736d6470ec5b7b7230
root@test-pg:/home/adm5# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
root@test-pg:/home/adm5# docker run --name pg-server --network pg-local -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
Unable to find image 'postgres:15' locally
15: Pulling from library/postgres
52d2b7f179e3: Pull complete 
d9c06b35c8a5: Pull complete 
ec0d4c36c7f4: Pull complete 
aa8e32a16a69: Pull complete 
8950a67e90d4: Pull complete 
1b47429b7c5f: Pull complete 
a773f7da97bb: Pull complete 
7bddc9bbcf13: Pull complete 
60829730fa39: Pull complete 
f3d9c845d2f3: Pull complete 
cfcd43fe346d: Pull complete 
576335d55cdb: Pull complete 
caad4144446c: Pull complete 
Digest: sha256:a5e89e5f2679863bedef929c4a7ec5d1a2cb3c045f13b47680d86f8701144ed7
Status: Downloaded newer image for postgres:15
0a64a5b352acac26adf57facf0491ec0e5af5ec2e3919ef5d22be7c6f871b121
docker: Error response from daemon: network pg-local not found.
root@test-pg:/home/adm5# docker network create pg-local
b0f2e1406d3b71605148e0af404d5b1d70064b0e53abbaaf8ee072bb0f4a9850
root@test-pg:/home/adm5# docker run --name pg-server --network pg-local -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
docker: Error response from daemon: Conflict. The container name "/pg-server" is already in use by container "0a64a5b352acac26adf57facf0491ec0e5af5ec2e3919ef5d22be7c6f871b121". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
root@test-pg:/home/adm5# docker rm 0a64a5b352acac26adf57facf0491ec0e5af5ec2e3919ef5d22be7c6f871b121
0a64a5b352acac26adf57facf0491ec0e5af5ec2e3919ef5d22be7c6f871b121
root@test-pg:/home/adm5# docker run --name pg-server --network pg-local -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
ea900b1490c58e03de191bf30ce9fa31e4035a921b3fa2f299b67d515e668b2a
root@test-pg:/home/adm5# docker run -it --rm --network pg-local --name pg-client postgres:15 psql -h pg-server -U postgres
Password for user postgres: 
psql (15.4 (Debian 15.4-1.pgdg120+1))
Type "help" for help.

postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# \l
                                                List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges   
-----------+----------+----------+------------+------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/postgres          +
           |          |          |            |            |            |                 | postgres=CTc/postgres
 test      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | 
(4 rows)

test=# select * from one;
 i  
----
 10
 20
(2 rows)

test=# \q
root@test-pg:/home/adm5# exit
exit
adm5@test-pg:~$ exit
logout
Connection to 51.250.92.158 closed.
yar@home .ssh % psql test -h 51.250.92.158 -U postgres
Password for user postgres: 
psql (14.8 (Homebrew), server 15.4 (Debian 15.4-1.pgdg120+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

test=# select * from one;
 i  
----
 10
 20
(2 rows)

test=# \q
yar@home .ssh % 

```
    
