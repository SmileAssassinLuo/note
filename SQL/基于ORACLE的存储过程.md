#### 存储过程

类似方法，可有参可无参，可有返回值。

SQL 会多次连接和关闭数据库，通过存储过程，将业务逻辑存储起来，可供程序复用，降低资源请求的消耗。

示例：主要介绍 入参和带返回值

```plsql
CREATE OR REPLACE P_GETSALWITHID (I_EMPNO IN EMP.EMPNO%TYPE,O_SAL OUT EMP.SAL%TYPE) AS
--不需要DECLARE，可直接声明变量
	V_SAL EMP.SAL%TYPE ;  -- 未用到此变量
BEGIN
	
	SELECT SAL INTO O_SAL FROM EMP WHERE EMPNO = I_EMPNO ;
	
END;
```



#####  调用存储过程

* plsql develop 工具调用
* 本地程序 cmd
* JAVA 等程序







