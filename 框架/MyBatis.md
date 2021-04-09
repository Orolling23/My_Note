# MyBatis

## #{}和${}的区别

* ${}是Properties文件中的变量占位符，用于标签属性值和sql内部，**属于静态文本替换**
* #{}是SQL中的参数占位符，MyBatis会将sql中的#{}替换成 ?，在SQL执行前使用PreparedStatement的参数设置方法，替换参数值。

一般情况下使用#{}，可以一定程度上防止SQL注入  

## Xml 映射文件中，除了常见的 select|insert|update|delete 标签之外，还有哪些标签？

还有很多其他的标签，`<resultMap>`、`<parameterMap>`、`<sql>`、`<include>`、`<selectKey>`，加上动态 sql 的 9 个标签，`trim|where|set|foreach|if|choose|when|otherwise|bind`等，其中为 sql 片段标签，通过`<include>`标签引入 sql 片段，`<selectKey>`为不支持自增的主键生成策略标签。

## Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗

* Dao接口，也就是常说的Mapper接口。Mapper接口是没有实现类的，当调用接口方法时，接口全限定名 + 方法名拼接字符串作为key值，可唯一定位一个`MapperedStatement`。在 MyBatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象。

* Dao接口的工作原理是JDK动态代理，MyBatis运行时会使用JDK动态代理为DAO生成代理Proxy对象，Proxy会拦截接口方法，转而执行MappedStatement所代表的SQL，然后将SQL执行结果返回。 
* 方法可以重载，但是XML中的ID不允许重复。**Mybatis 的 Dao 接口可以有多个重载方法，但是多个接口对应的映射必须只有一个，否则启动会报错。**