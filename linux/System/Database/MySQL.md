## 1.用户管理

```mysql
- 新建用户
> create user 'username'@'%' identified by 'password';

- 分配权限
> grant privileges on databasename.tablename to 'username'@'host';
```

