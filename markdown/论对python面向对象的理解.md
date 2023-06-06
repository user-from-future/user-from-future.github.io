面对对象都是先定义类和创建对象，我们先说说怎么定义类吧。

### **定义类**

类是用来描述具有相同属性和方法的对象集合。就像人可以通过不同的肤色划分为不同的种族，食物也有不同的种类，商品也是形形色色。但划分为同一类的物体， 肯定具有相似的特征和行为方式。  
  
举一个简单的例子。对于同一款自行车而言，它们的组成结构都是一样的， 如车架、车轮和脚踏板等。那么我们就可以通过Python可以定义这个自行车的类:  

```python
class Bike:

compose = ['frame', 'wheel', 'pedal']
```

  
通过使用class定义一个 自行车的类，类中的变量compose称为类的变量，专业术语为类的属性。这样，顾客购买的自行车组成结构就是一样的了 。  

```python
my_ bike = Bike()

you bike = Bike()

print (my_ bike. compose)

print (you bike . compose)     //#类的属性都是一样的
```

  
  
在左边写上变量名，右边写上类的名称，这个过程称之为类的实例化，而my bike就是类的实例。通过“”加上类的属性，就是类属性的引用。类的属性会被类的实例共享，所以结果都是样的。  

实例方法

我们引申一下，关于python的实例方法，方法就是函数，方法是对实例进行使用的，所以又叫实例方法。

我们还用自行车举例，对于自行车而言，它的方法就是骑行。

```python
class Bike:

    compose = ['frame', 'wheel', 'pedal']

    def use(self)
        print('you are riding')

my_bike=bike()
my_bike.use()


>>>运行结果
>>>you are  riding
```

**_这里的self参数就是实例的本身。_**

**和函数一样，实例方法也是可以有参数的。**

**下面还是用自行车举例：**

```python
class Bike:

    compose = ['frame', 'wheel', 'pedal']

    def use(self,time)
        print('you ride {}m'.format(time))

my_bike=bike()
my_bike.use(10)


>>>运行结果
>>>you ride 10 m
```

python的类中有一些“魔法方法”——\_init\_\(\),这些在我们创造实例的时候，不需要引用该方法也是可以被自动执行。

到这里，我们就会发现，python的面向对象就这么简单，相信你看完这些例子就能明白，希望大家可以喜欢。