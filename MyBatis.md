# MyBatis 笔记

[中文文档](http://www.mybatis.org/mybatis-3/zh)

### (一) 入门知识

① 构建 SqlSessionFactory

② 从 SqlSessionFactory 中获取 SqlSession，通过 SqlSession 实例来直接执行已映射的 SQL 语句

③ SQL 语句映射是如何进行的

1. 基于 XML 的映射语言来实现

2. 基于Java 注解来实现

   > 对于简单语句来说，注解使代码显得更加简洁，然而 Java 注解对于稍微复杂的语句就会力不从心并且会显得更加混乱。因此，如果你需要做很复杂的事情，那么最好使用 XML 来映射语句。

### (二) 进阶知识

① 作用域（Scope）和生命周期

1. SqlSessionFactoryBuilder

   这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。因此 SqlSessionFactoryBuilder 实例的最佳作用域是**方法作用域**（也就是局部方法变量）。你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但是最好还是不要让其一直存在以保证所有的 XML 解析资源开放给更重要的事情。

2. SqlSessionFactory

   SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由对它进行清除或重建。使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏味道（bad smell）”。因此 SqlSessionFactory 的最佳作用域是**应用作用域**。有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

3. SqlSession

   每个线程都应该有它自己的 SqlSession 实例。**SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域**。绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。也绝不能将 SqlSession 实例的引用放在任何类型的管理作用域中，比如 Servlet 架构中的 HttpSession。如果你现在正在使用一种 Web 框架，要考虑 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。换句话说，**每次收到的 HTTP 请求，就可以打开一个 SqlSession，返回一个响应，就关闭它**。这个关闭操作是很重要的，你应该把这个关闭操作放到 finally 块中以确保每次都能执行关闭。下面的示例就是一个确保 SqlSession 关闭的标准模式：

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  // do work
} finally {
  session.close();
}
```

​	在你的所有的代码中一致性地使用这种模式来保证所有数据库资源都能被正确地关闭。

4. 映射器实例（Mapper Instances）

   映射器是一个你创建来绑定你映射的语句的接口。映射器接口的实例是从 SqlSession 中获得的。因此从技术层面讲，任何映射器实例的最大作用域是和请求它们的 SqlSession 相同的。尽管如此，映射器实例的最佳作用域是**方法作用域**。也就是说，映射器实例应该在调用它们的方法中被请求，用过之后即可废弃。并不需要显式地关闭映射器实例，尽管在整个请求作用域（request scope）保持映射器实例也不会有什么问题，但是很快你会发现，像 SqlSession 一样，在这个作用域上管理太多的资源的话会难于控制。所以要保持简单，最好把映射器放在方法作用域（method scope）内。下面的示例就展示了这个实践：

```java
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // do work
} finally {
  session.close();
}
```

② 数据库映射模式

1. 数据库都具备良好的第三范式或 BCNF 范式

③ 映射方式

1. 一个更通用的方式来执行映射语句是使用映射器类。一个映射器类就是一个仅需声明与 SqlSession 方法相匹配的方法的接口类;MyBatis 3 构建在全面且强大的基于 Java 语言的配置 API 之上；这个配置 API 是基于 XML 的 MyBatis 配置的基础，也是新的基于注解配置的基础。注解提供了一种简单的方式来实现简单映射语句，而不会引入大量的开销。