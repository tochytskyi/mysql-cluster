# mysql-cluster
Set-up simple mysql master slave replication: master and 2 slaves

### Start docker
```shell
./build.sh
```

### Check replication is working for each slave
1. Check that there are not tables in slaves
```shell
docker exec mysql_slave_1 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SHOW TABLES \G'"
```
```shell
docker exec mysql_slave_2 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SHOW TABLES \G'"
```
2. Create a table in master
```shell
docker exec mysql_master sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; CREATE TABLE IF NOT EXISTS data(task_id INT AUTO_INCREMENT PRIMARY KEY, title VARCHAR(255) NOT NULL)'"
```
3. Check again that there are the table `data` in slaves
```shell
 docker exec mysql_slave_1 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SHOW TABLES \G'"                   

*************************** 1. row ***************************
Tables_in_mydb: data
```
```shell
 docker exec mysql_slave_2 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SHOW TABLES \G'"                   

*************************** 1. row ***************************
Tables_in_mydb: data
```
4. Insert some rows into `data` table in master
```shell
docker exec mysql_master sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; INSERT INTO data (title) VALUES (\"Example_1\"), (\"Example_2\")'"
```
5. Check `data` table in slaves
```shell
docker exec mysql_slave_1 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SELECT * FROM data'"
task_id title
1       Example_1
2       Example_2
```
```shell
docker exec mysql_slave_2 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SELECT * FROM data'"
task_id title
1       Example_1
2       Example_2
```

### Check replication is working after turning off a slave
```shell
docker-compose stop mysql_slave_1
```

Insert more data to master
```shell
docker exec mysql_master sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; INSERT INTO data (title) VALUES (\"Example_3\"), (\"Example_4\")'"
```

Turn on slave
```shell
docker-compose up -d mysql_slave_1
```

Check that data was replicate from the lost period of time
```shell
docker exec mysql_slave_1 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SELECT * FROM data'"
task_id title
1       Example_1
2       Example_2
3       Example_3
4       Example_4
```

### Try to remove a column in database on slave node
Delete column `title` from the first slave
```shell
docker exec mysql_slave_1 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; ALTER TABLE data DROP COLUMN title'"
docker exec mysql_slave_1 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SELECT * FROM data'"

task_id
1
2
3
4
```

Insert more data to master and check slave. New data replicated to the remaining column
```shell
docker exec mysql_master sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; INSERT INTO data (title) VALUES (\"Example_5\")'"
docker exec mysql_slave_1 sh -c "export MYSQL_PWD=111; mysql -u root -e 'USE mydb; SELECT * FROM data'"                            

task_id
1
2
3
4
5
```