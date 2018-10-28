# Run
```
alias d=docker
alias dc=docker-compose
dc up -d
```


# Things to try out
```
dc exec -T -upostgres pub psql <<EOF
insert into foo values
(1),
(2);
EOF

# here, sub MUST be able to connect to pub
dc exec -upostgres sub psql -c "CREATE SUBSCRIPTION mysub CONNECTION 'dbname=postgres host=pub user=postgres password=password' PUBLICATION mypub;"
dc exec -upostgres sub psql -c "SELECT * FROM foo"
```

## publisher goes away
```
dc stop pub
```
note the error messages in `dc logs -f sub`:
```
could not connect to the publisher: could not translate host name "pub" to address: Temporary failure in name resolution
```
but subscriber can still query the old data
```
dc exec -upostgres sub psql -c "SELECT * FROM foo"
```
then publisher comes back
```
dc start pub
```

## subscriber goes away
```
dc stop sub
```
publisher keeps writing
```
dc exec -T -upostgres pub psql <<EOF
insert into foo values
(3),
(4);
EOF
```
sub comes back
```
dc start sub
dc exec -upostgres sub psql -c "SELECT * FROM foo"
```


# Cleanup
```
dc down
d volume rm $(d volume ls -qf label=de.felixhummel.project=postgres-logical-replication)
```

