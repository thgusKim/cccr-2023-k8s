# MySQL Read Replication

```bash
kubectl create -f .
```

```bash
kubectl run dbclient --image ghcr.io/c1t1d0s7/network-multitool -it --rm
```

```bash
# host mydb-0.mydb
# host mydb-1.mydb
# host mydb-read
```

```bash
# mysql -h mydb-0.mydb -u root -e 'CREATE DATABASE mydb'
# mysql -h mydb-0.mydb -u root -e 'CREATE TABLE mydb.mytb (message VARCHAR(100))'
# mysql -h mydb-0.mydb -u root -e 'INSERT INTO mydb.mytb VALUES ("hello workd")'
```

```bash
# mysql -h mydb-1.mydb -u root -e 'SELECT * FROM mydb.mytb'
```

```bash
# exit
```

```bash
kubectl scale sts mydb --replicas=3
```

```bash
kubectl run dbclient --image ghcr.io/c1t1d0s7/network-multitool -it --rm
```

```bash
# mysql -h mydb-2.mydb -u root -e 'SELECT * FROM mydb.mytb'
```



