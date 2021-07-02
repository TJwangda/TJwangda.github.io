## python3基础

+ **数据类型**：列表[val1,val2]  、元组(val1,val2)  、字典{key:value,key2:value}

+ **不可变类型**：数字、字符串、元组是不可变的

  对不可变类型的变量重新赋值，实际上是重新创建一个不可变类型的对象，并将原来的变量重新指向新创建的对象（如果没有其他变量引用原有对象的话（即引用计数为0），原有对象就会被回收）。

+ **可变类型**：列表、字典是可变的

+ **多值传参**  参数名前加一个\*号可以接元组  两个*号可以接字典。

+ **拆包** ，调用多值传参时加一个或两个星号，简化传参，直接传递元组或字典。否则需要把元组或字段拆分成(val1,val2,val3,key1 = val4,key2 = val5)才能传参。

+ **类** 定义类方法，第一个参数必须是self(哪个对象调用方法，self就是哪个对象的引用)。

  1. 创建对象步骤：（1）、给对象在内存分配空间，创建对象。（2）、对象属性初始值初始化，初始化方法（__ init__），对象的内置方法（内置方法都是变量名前后两个下划线）。

  2. **__ new__方法**：对象创建前调用， 分配内存空间。返回内存空间地址给\__init__方法。

  3. **__ name__** 方法：相当于java的main方法

     ~~~python
     if __name__ == "__main__":
         # 执行本类测试代码，其他类调用时不会进入执行。
     ~~~

  4. **私有属性、私有方法：**定义方法，在属性名或方法名前加两个下划线

  5. **继承** 语法：class 类型(父类名)。**多继承** 语法：class 类型(父类名1，父类名2)

  6. **类方法** 语法：

     ~~~python
     @classmethod
     def 类方法名(cls):   # cls 类的引用，哪个调用就是哪个，不用传递。
     	pass
     ~~~

  7. 静态方法

     ~~~python
     @staticmethod
     def 类方法名():   # 不访问类属性，也不访问实例属性。
     	pass
     ~~~

  8. python包，需要有__ init__.py的文件。

+ **身份运算符** is 判断两个变量引用的对象是否为同一个，==用于判断两个变量引用的值是否相等。


## 爬虫

  
