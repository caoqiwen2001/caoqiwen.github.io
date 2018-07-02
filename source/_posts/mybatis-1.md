---
title: Mybatis的基础配置
date: 2016-11-17 23:10:31
tags:
  - Java
categories:
    - Mybatis
type:
    - tags
---
# Mybatis的基础配置
  mybatis最好的优点就是采用面向接口的方法来实现数据库的增删查改，很方便的对数据库进行相关的操作，开篇讲如何配置mybatis。
#   配置相关选项
1.   下载mybatis的相关包，本项目中用的是mybatis3.0 。
2.   配置数据库选项

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 对事务的管理和连接池的配置 -->
    <properties resource="db.properties"></properties>
    <environments default="development">
        <environment id="users">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="${db.driver}" />
                <property name="url" value="${db.url}" />
                <property name="username" value="${db.username}" />
                <property name="password" value="${db.password}" />
            </dataSource>
        </environment>
    </environments>

    <!-- mapping 文件路径配置 -->
    <mappers>
        <mapper resource="model/UserMapper.xml" />
    </mappers>
</configuration>
```
我把数据库的相关信息保存在db.propertise文件中，方便日后的维护工作。需要注意的是id的值，我们在测试的过程中会用到。
3. 创建对应的映射文件UserMapper.xml,将对应sql语句写在这个文件中，项目中一般一个模块的sql都放在该模块中。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="model.UserMapper">
	<select id="selectUser" resultType="model.User">
		select *from User where
		id=#{id}
	</select>
<!-- parameterType 代表插入数据类型为user类型 -->
	<insert id="addUser" parameterType="String">
		insert into user(userName,userPwd) values (#{userName},#{userPwd})
	</insert>

</mapper>
```
注意这里的namespace要对应的包下面的类名。对于查询语句，这里返回的是User类，而插入语句， 传入的参数类型为String类型，对于返回类型这里没有做相关操作。
4. 创建对应的model类
所谓的model，表示对应数据库中相应的属性名，这里创建一个User类

```
package model;

public class User {
	private int id;
	private String userName;

	@Override
	public String toString() {
		return "User [id=" + id + ", userName=" + userName + ", userPwd=" + userPwd + "]";
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public String getUserPwd() {
		return userPwd;
	}

	public void setUserPwd(String userPwd) {
		this.userPwd = userPwd;
	}

	private String userPwd;
}

```
有了这些前期工作准备就可以开始查询工作了。可以创建一个单元测试来测试对应方法。
## 关于使用接口的方法来进行数据库操作
接口的方式省去了手动写代码的工作量。值得注意的是在配置对应的接口命名空间及对应cfg文件的配置。如下图所示：
创建一个UserMapper接口类：

```
public interface UserMapper {
	public User selectUser(int id);
	public List<User> selectAllUser();
	public void addUser(User user);
	public void updateUser(User user);
	public void deleteUser(int id);

}

```
该类与我们的UserMapper.xml相对应。值得注意的是在配置mybatis时，不需要加class类的路径，即如下所示：
```
  <!-- mapping 文件路径配置 -->
    <mappers>
       <mapper resource="model/UserMapper.xml" />
     <!--    <mapper class="dao.UserMapper"/>-->
    </mappers>
```
