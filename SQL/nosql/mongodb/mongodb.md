#### Mongodb

##### 一，安装

官网下载对应操作系统版本的exe，`windows` 傻瓜式安装即可；

##### 二，设置数据库账户密码

在`mongodb`安装目录，`D:\softPath\mongoDB\bin`的 `bin`目录下，打开 `mongo.exe`,输入如下命令

1. `use admin` 进入`admin`数据库

2.  `db.createUser({user:'admin',pwd:'password',roles:[{role:'root',db:'admin'}]})` 设置超级管理员角色账户。

3. `Successfully added user: { "user" : "admin",  "roles" : [ {  "role" : "root",  "db" : "admin" } ] }` 弹出上述语句，则表示成功创建超级管理员角色账户。

4. 设置完成，可以通过指令 `show users` 查看是否设置成功

5. 开启权限验证

   找到`MongoDB`安装目录下的`bin`目录中的`mongod.cfg`文件，开启权限验证功能：

   `security:
     authorization: enabled`

   重启`mongdb`

6. 登录

   *  方式一： `use admin` ,`db.auth('admin', 'password')` ;
   *  方式二 ：`mongo admin -u admin -p password`;

7. 除了设置超级管理员账号以外，还可以为每个数据库单独设置账号。

   ```
   use myMongoDB  // 跳转到需要添加用户的数据库,myMongoDB:数据库名字
   
   db.createUser({
     user: 'tao',          // 用户名
     pwd: 'Abc123++',      // 密码
     roles:[{
       role: 'readWrite',  // 读写权限角色
       db: 'myMongoDB'     // 数据库名
     }]
   })
   
   
   ```

   登录: `mongo myMongoDB -u tao -p Abc123++`

8. 常用其他命令

   ```
   show users  // 查看当前库下的用户
   db.dropUser('testadmin')  // 删除用户
   db.updateUser('admin', {pwd: '654321'})  // 修改用户密码
   db.auth('admin', '654321')  // 密码认证
   ```

#### 三，mongodb 常用操作

###### 一，启动 数据库

`dos`命令：

`D:\softPath\mongoDB\bin>mongod --dbpath D:\softPath\mongoDB\data`

###### 二，闪退问题

在任意目录新建`data`文件夹，`data`下新建`db`和`log`文件夹, 执行`dos`命令：

`D:\softPath\mongoDB\bin>mongod --dbpath c:D:\softPath\mongoDB\data`

###### 三，增删改查



