# mybatis 入门学习

## 1. Mybtatis CRUD

* ###Mybatis的配置.xml文件中标签的`parameterType`属性的使用方法

  在标签体通过#{}来使用方法的参数

  * 若参数是基本类型，所以#{}其中可以填任何名称
  * 若参数类型非基本类型，则#{}中填的是相应的由setter与getter形成的属性名此处我们省略了表示参数类型的名称，实际上完整的内容应填#{随意的名称.相应的属性名}

* ### Save操作后将保存到数据中的id返回给传入时的user中

  * ```xml
    <selectKey resultType="int" keyColumn="id" keyProperty="id"  order="AFTER" >
                select last_insert_id()
     </selectKey>
    ```

  * 其中`resultType`为该语句的返回值类型

  * `keyColumn`为数据库中的对应列名

  * `keyProperty`为要返回值将要保存的属性的属性名

  * `order`为执行该保存过程的顺序

* ### 接下来

* 