# MySQL Pre-Requisite

Check and confirm the user and permissions use default password `D4BB3A9AFC45D210`
```
kubectl exec -it -n datahub mysql-0 -- mysql -u root -p;
```

If plugin is auth_socket, switch it:
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
FLUSH PRIVILEGES;
```

Try logging in again with:
```
kubectl exec -it -n datahub mysql-0 -- mysql -u root -p
```
