> PL/SQL：可以直接通过PL/SQL完成简单的业务逻辑，具有编程语言的语法结构

**DECLARE** ----用于声明变量、游标

```plsql
DECLARE 
--CONSTANT不加即为常用，不加就是变量
变量名[是否常量CONSTANT] 数据类型 [是否非空] [:==初始值]
--变量名 表名.列名%type :这是根据指定字段名的类型给变量指定类型
v_s_sex student.s_sex%TYPE;
--类似创建了一个对象
v_student student%rowtype;
```

**BEGIN**---------程序的开始

```plsql
BEGIN
--||为连接符
SELECT s_name into v_s_name FROM student where s_id=v_s_id;
dbms_output.put_line ('id为'||v_s_id||'的同学的姓名为：'||v_s_name );
--这个过程类似调用set方法给对象赋值
select s_name,s_birth,s_sex into v_student.s_name,v_student.s_birth,v_student.s_sex from student where s_id=3;
--类似调用get方法取出值
dbms_output.put_line ( '姓名：'||v_student.s_name );
dbms_output.put_line ( '性别：'||v_student.s_sex );
dbms_output.put_line ( '生日：'||v_student.s_birth );
```

**EXCEPTION**--处理异常

| 预定义异常          | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| access_into_null    | 试图给一个没有初始化的对象赋值                               |
| case_not_found      | 在CASE语句中没有WHEN自居被选中，并且没有ELSE子句             |
| invalid_number      | 试图将一个非有效的字符串转换成数字                           |
| loggin_denied       | 使用无效的用户名和扣了登陆oracle                             |
| no_data_found       | 查询语句无返回数据，伙子引用了一个删除的元素，伙子引用了一个没有被初始化的元素 |
| timeout_on_resource | oracle在等待资源时发送的超时现象                             |

```plsql
DECLARE
自定义异常名 exception;
BEGIN
if 条件 then
	raise 自定义异常名;
end if;
EXCEPTION
 when v_exp then
  dbms_output.put_line ('要提示的信息');
END;

--案例
DECLARE
v_exp exception;
BEGIN
for num in 1..10 loop
		  dbms_output.put_line (num);
		if num=4 then
				raise v_exp;
		end if;
end loop;
EXCEPTION
 when v_exp then
  dbms_output.put_line ('数据为4，异常抛出');
END;
```

**END**------------结束

> if条件语句

```plsql
--IF语法结构：
if 条件 then
执行的语句
else
执行的语句
end if

--案例
DECLARE
v_count NUMBER(11);
BEGIN
SELECT COUNT(1)  into v_count from student where s_sex='男' group by s_sex;
if v_count>0 then
dbms_output.put_line ( '有男的' );
else
dbms_output.put_line ( '没有男的' );
end if;
END;
```

> case

```plsql
CASE 用来判断的列名
	WHEN 当值是什么的时候 THEN
		就显示的列的名字
	ELSE
		就显示的列的名字;
END CASE;
```

> loop

```plsql
--LOOP 语法，如果没有if语句去终止循环，就是一个死循环
LOOP
	要执行的语句
	IF 条件 THEN
	要执行的语句
		EXIT; 
	END IF; 
END LOOP;

--案例
DECLARE
	num_lmz NUMBER(11) :=1;
BEGIN
LOOP
	 dbms_output.put_line ('第 '||num_lmz||' 次循环');
	 num_lmz:=num_lmz+1;
	IF num_lmz>1000 THEN
		EXIT;
	END IF; 
END LOOP;
END;
```

> whilie loop

```plsql
--while loop语法
while 条件 loop
	执行的语句
end loop;

--案例
DECLARE
	num_lmz NUMBER(11) :=1;
BEGIN
	while num_lmz<1000 loop
	dbms_output.put_line ('第 '||num_lmz||' 次循环');
	 num_lmz:=num_lmz+1;
	 end loop;
END;
```

> for loop

```plsql
--for loop语法
for 变量名 in 最小循环次数..最大循环次数 loop
	dbms_output.put_line ('第 '||变量名||' 次循环');
end loop;
	 
--案例
DECLARE
BEGIN
	for num_lmz in 1..100 loop
	dbms_output.put_line ('第 '||num_lmz||' 次循环');
	end loop;
END;
```

#### 存储过程

**什么是函数？**
函数是类似java方法的模块，可以传入参数，可以有返回值，可以在函数内进行一系列的计算和sql执行，一次创建永久可以使用，除非删除，能极大提高效率。

**什么是存储过程？**
存储过程是一系列sql执行的集合，可以说是和函数是兄弟，可以有参数，可以有返回值，一次创建永久可以使用，除非删除。

**区别？**
函数和存储过程的区别：

- 函数必须只有一个返回值，不能没有，而存储过程可以没有或者有多个参数
- 函数一般用来返回处理（计算）后的数据，而存储过程一般用来去执行sql语句，操作数据库表，流程控制。
- 函数用function，存储过程声明用procedure。
- 函数必需要返回类型，而存储过程不需要。
- 函数不能单独做为pl/sql去运行，必须是表达式的一部分，而存储过程可以单独运行，如果有返回值的话也必须是pl/sql中的一部分，没有就不用。
- 函数既可以使用out或者in/out来返回值，也可以使用retrun来返回值，但是存储过程只能out或者in/out。
- DML中不能调用存储过程，但是可以调用函数。

**共同点**：方便高效，一次创建都可以无限调用。

**适用场景**：
函数一般在只有一个返回值的时候使用，并且在DML里只能调用函数，不能调用存储过程
存储过程在没有或者多个返回值的时候使用，并且不能在DML里调用

**存储过程案例**

```plsql
--创建存储过程
create or replace procedure 过程名(参数 in 参数类型... 还可以有 参数名 out 参数类型) is
begin
要执行的主体
end;

--调用并输出返回值
DECLARE
	num4 number(10);
BEGIN
	lmz02(10,20,30,num4);
	 dbms_output.put_line (num4);
end;
```

**函数的调用**

```plsql
create or replace FUNCTION lmz03(num01 in NUMBER,num02 in NUMBER) return NUMBER as
num03 NUMBER;
begin
num03:=num01+num02;
return num03;
end;

```

#### 游标

简单的来说就是一个数组或者list集合

```plsql
DECLARE
	cursor 游标名[可选参数] is select....;
	变量名 使用原表类型%rowtype;
BEGIN
	open 游标名;
	loop
	fetch 游标名 into 变量名;
	exit when 游标名%notfind;
	end loop;
	close 游标名;
end;

--如果使用的是for in循环
DECLARE
	cursor 游标名[可选参数] is select....;
BEGIN
	for stu in 游标名 loop
	dbms_output.put_line('姓名：'||stu.S_NAME);
	end loop;
end;

--单独取变量
declare
    v_s_name student.S_NAME%type;
    v_s_sex  student.S_SEX%type;
    cursor lmz is select *
                  from student;
begin
    for stu in lmz
        loop
            v_s_name := stu.S_NAME;
            v_s_sex := stu.s_sex;
            dbms_output.put_line('姓名：' || v_s_name || '性别:' || v_s_sex);
        end loop;
end;
```

> 伪列

rowid、rownum

```plsql
select STUDENT.S_ID, STUDENT.S_NAME, STUDENT.S_SEX, STUDENT.S_BIRTH, ROWNUM
                  from student
                  where ROWNUM between rn01 and rn02;
```

> 分页

```plsql
--创建分页存储过程
create or replace procedure pageData(pageIndex number, pageSize number) is
    rn01 number := (pageIndex - 1) * pageSize;
    rn02 number := pageIndex * pageSize;
    cursor stu is select STUDENT.S_ID, STUDENT.S_NAME, STUDENT.S_SEX, STUDENT.S_BIRTH, ROWNUM
                  from student
                  where ROWNUM between rn01 and rn02;
begin
    DBMS_OUTPUT.PUT_LINE('rn01:' || rn01 || ' rn02:' || rn02);
    DBMS_OUTPUT.PUT_LINE('第 ' || pageIndex || '页--每页数量：' || pageSize);
    FOR ha in stu
        loop
            DBMS_OUTPUT.PUT_LINE(
                        'rownum:' || ha.ROWNUM || ' id' || ha.S_ID || ' 姓名： ' || ha.S_NAME || ' 性别： ' || ha.S_SEX ||
                        ' 生日： ' || ha.S_BIRTH);
        end loop;
end;

--调用存储过程，并传入pageIndex和pageSize
call pageData(1, 5);
```

