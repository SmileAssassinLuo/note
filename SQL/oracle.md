#### Oracle





#### PLSQL

PLSQL 是Oracle公司在SQL基础上进行扩展而成的一种过程语言。PLSQL提供了典型的高级语言特 性，包括封装，例外处理机制，信息隐藏，面向对象等；并把最新的编程思想带到了数据库服务器和工具 集中。

在SQL中添加了过程处理语句（分支，循环等），使得SQL语言具有过程处理能力。

##### PLSQL的优势

PLSQL无异于是有优势的，其本质的特性就是直接用sql进行编程进行业务逻辑的处理，而不像java,像c++那种写个查询语句还需要封装成一个statement语句去执行 。如图：

![PLSQL](D:\work\note\SQL\img\PLSQL.png)



##### PLSQL中块的概念

块是PLSQL中执行的一个小单位，分三种 匿名块、存储过程、函数。

其中匿名块是没有名字的，存储过程没有返回值，函数有名字有返回值。

匿名块其块的语法为：

程序结构：

```plsql
--DECLARE:用来申明变量 相当于JAVA中的初始化变量 ，Optional：可选的，无变量或游标；
--BEGIN:是一段程序的开始，可以看成一个方法。Mandatory：必要的；
--EXCEPTION：用来处理逻辑中可能出现的异常
--END;是一个结束。一个小方法结束了。


DECLARE (Optional)
Variables, cursors, user-defined exceptions
BEGIN (Mandatory)
	SQL statements
	PL/SQL statements
EXCEPTION (Optional)
Actions to perform when errors occur
END; (Mandatory)
```

其他块的对比：

![plsql块](D:\work\note\SQL\img\plsql块.jpg)

##### 打印 HELLO WORLD

```plsql
DECLARE   --若无变量，可省
	I INTEGER;
BEGIN
	--类似js:console.log();
	DBMS_OUTPUT.PUT_LINE('HELLO WORLD');
END;
```



##### PLSQL 定义变量

pl/sql的变量分为两大类：

* 普通数据类型（char,varchar2,number,boolean,long,date）
* 特殊变量类型（引用型变量，记录型变量）

###### 普通变量

定义的方法为：变量名 变量类型（数据的长度） Not null ：=默认值 后面的都是可选项。

```plsql
DECLARE
 v_hiredate DATE;
 v_deptno NUMBER(2) NOT NULL := 10;
 v_location VARCHAR2(13) := 'Atlanta';
 c_comm CONSTANT NUMBER := 1400;
 
```

赋值的方式有两种

* 直接用语句赋值  v_deptno NUMBER(2)  := 10;
* 语句赋值，select   ...   into   ...  ;（select 值 to 变量，一般用于从哪里计算或者查询后）；

```plsql
-- 示例：打印个人信息，包括姓名，薪水，地址等
DECLARE
	--姓名
	v_name VARCHAR2(20) := 'Isaiah';
	--薪水
	v_sal NUMBER;
	--地址
	v_addr VARCHAR2(200);
	
BEGIN
	--直接赋值
	v_sal := 3000;
	--语句赋值
	SELECT '深圳市xxx' INTO v_addr FROM DAUL;
	
	--打印变量
	DBMS_OUTPUT.PUT_LINE('姓名：'||v_name||',薪水：'||v_sal||'，地址：'||v_addr);
END;
	

```



###### 引用型变量：

变量的类型和长度取决于表中字段的类型和长度。

定义方式：通过**表名.列名%TYPE**指定变量的类型和长度，例如 ：

```PLSQL
DECLARE
	V_NAME STUDENTS.NAME%TYPE ;
```

示例：查询编号为7777的员工姓名和薪水，并打印。

```PLSQL
DECLARE
	V_NAME EMP.ENAME%TYPE ;
	V_SAL EMP.SAL%TYPE ;
BEGIN
	SELECT ENAME,SAL INTO V_NAME,V_SAL FROM EMP WHERE ID = 7777;
	--
	DBMS_OUTPUT.PUT_LINE('姓名：'||V_NAME||',薪水：'||V_SAL);
END;
```

引用型变量的优势就在于，可以不用预先知道变量的类型和长度，也能在表中数据更改类型和长度后动态保证SQL灵活运行。



###### 记录型变量

获取的是一整条记录，类似对象，通过点语法取值。

优点：简化语句，不用定义多个字段；

缺点：当所需字段过少时，查询 *  性能不高；

```plsql
DECLARE
	V_EMP EMP%ROWTYPE ;
BEGIN
	SELECT * INTO V_EMP FROM EMP WHERE ID = 7777 ;
	DBMS_OUTPUT.PUT_LINE('姓名：'||V_EMP.NAME||',薪水：'||V_EMP.SAL);
END;
```



##### 条件语句

```plsql
DECLARE 
	V_COUNT NUMBER;
BEGIN
	SELECT COUNT(1) INTO V_COUNT FROM EMP ;
	IF V_COUNT > 20 THEN 
		DBMS_OUTPUT.PUT_LINE('COUNT IS OVER 20.AND COUNT IS '||V_COUNT);
	ELSIF V_COUNT >= 10 THEN
		DBMS_OUTPUT.PUT_LINE('COUNT IS XXXX.AND COUNT IS  '||V_COUNT);
	ELSE
		DBMS_OUTPUT.PUT_LINE('COUNT IS LESS 10 .AND COUNT IS  '||V_COUNT);
	END IF;
```

##### 循环语句

```plsql
--定义循环变量,初始值为1；
DECLARE 
	V_NUM NUMBER := 1 ;
BEGIN
	LOOP
		--定义退出循环条件
		EXIT WHEN V_NUM >10 ;
		DBMS_OUTPUT.PUT_LINE(V_NUM);
		--变量自增
		V_NUM := V_NUM + 1 ;
	END LOOP
END;
```



##### 游标

用于临时存储一个查询返回的多行数据，（类似于JAVA中的ResultSet集合），通过遍历游标，可以逐行访问处理该结果集的数据；

方式：声明  --> 打开  --> 读取  --> 关闭

###### 语法

* 游标声明：CURSOR  游标名(参数列表)  IS  查询语句

* 打开：OPEN   游标名
* 读取:   FETCH 游标名 INTO 变量列表
* 关闭：CLOSE 游标名

###### 游标的属性

| 游标属性  | 返回值类型 | 说明                                        |
| --------- | ---------- | ------------------------------------------- |
| %ROWCOUNT | 整型       | 获得FETCH返回的数据行数。                   |
| %FOUNT    | 布尔型     | 最近的FETCH语句返回一行数据为真，否则为假。 |
| %NOTFOUNT | 布尔型     | 与%FOUNT的返回值相反。                      |
| %ISOPEN   | 布尔型     | 游标打开时为真，否则为假。                  |

%NOTFOUNT 在游标中找不到数据为真，通常用于退出循环。

###### 创建和使用

无参游标

```plsql
--查询所有员工的姓名和薪水，并打印
DECLARE 
	-- 声明游标
	CURSOR C_EMP IS 
		SELECT ENAME,SAL FROM EMP ;
	-- 声明接收变量
	V_NAME EMP.ENAME%TYPE;
	V_SAL EMP.SAL%TYPE;

BEGIN
	-- 打开游标
	OPEN C_EMP;
	--循环遍历游标
	LOOP
		-- 获取游标的值，并赋值给变量
		FETCH C_EMP INTO V_NAME,V_SAL; 
		
		-- 找不到数据时，退出循环
		EXIT WHEN C_EMP%NOTFOUNT;
		
		-- 打印出姓名
		DBMS_OUTPUT.PUT_LINE(V_NAME || -- || V_SAL)
	
	END LOOP ;
                             
	-- 关闭游标
	CLOSE C_EMP;
	
END;


```

带参游标：在无参的基础上，声明时加参数，打开时加入条件选项，其他都一样；

```PLSQL
--查询某个部门员工的姓名和薪水，并打印
DECLARE 
	-- 声明游标
	CURSOR C_EMP(V_DEPNO EMP.DEPNO%TYPE) IS 
		SELECT ENAME,SAL FROM EMP WHERE DEPNO = V_DEPNO ;
	-- 声明接收变量
	V_NAME EMP.ENAME%TYPE;
	V_SAL EMP.SAL%TYPE;

BEGIN
	-- 打开游标
	OPEN C_EMP(10);
	--循环遍历游标
	LOOP
		-- 获取游标的值，并赋值给变量
		FETCH C_EMP INTO V_NAME,V_SAL; 
		-- 找不到数据时，退出循环
		EXIT WHEN C_EMP%NOTFOUNT;		
		-- 打印出姓名
		DBMS_OUTPUT.PUT_LINE(V_NAME || -- || V_SAL)
	END LOOP ;                           
	-- 关闭游标
	CLOSE C_EMP;	
END;
```

注意：FETCH 语句要在EXIT WHEN C_EMP%NOTFOUNT之前，不然会将最后一条数据打印2次。xxx%NOTFOUNT（默认false）

