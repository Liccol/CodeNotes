# JAVA 反射

JAVA反射机制是在运行状态中，对于任意一个类，都能够通过反射知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性。

这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

## 用途

在日常的第三方应用开发过程中，经常会遇到某个类的某个成员变量、方法或是属性是**私有的**或是**只对系统应用开放**，这时候就可以利用Java的反射机制通过反射来获取所需的私有成员或是方法。

当然，也不是所有的都适合反射，之前就遇到一个案例，通过反射得到的结果与预期不符。经过层层调用后在最终返回结果的地方对应用的权限进行了校验，对于没有权限的应用返回值是没有意义的缺省值，否则返回实际值，起到保护用户的隐私目的。

## 基本反射技术

1. 根据内容得到类

通过getClass()获得类名，通过getSuperClass()获取父类。

```
String name = "Huanglinqing";
Class c1 = name.getClass();
// 输出java.lang.String
System.out.println(c1.getName());
```

通过Class.forName(目标)获得目标类名

```
   String name = "java.lang.String";
   Class c1 = null;
   try {
          c1 = Class.forName(name);
          System.out.println(c1.getName());
      } catch (ClassNotFoundException e) {
  }
```

2. 获取Type属性

获得的内容都是Class类。

```
Class c1 = Boolean.TYPE;
Class c2 = Byte.TYPE;
Class c3 = Float.TYPE;
Class c4 = Double.TYPE;
```

## 获取类的成员

当类中**方法定义为私有**的时候我们能调用？不能！

当**变量是私有**的时候我们能获取吗？不能！

但是反射可以，比如源码中有你需要用到的方法，但是那个方法是私有的，这个时候你就可以通过反射去执行这个私有方法，并且获取私有变量。

举例：
```
public class Test {
	// 私有变量
    private int age;
    private String name;
    private int testint;
 
    public Test(int age) {
        this.age = age;
    }
 
    public Test(int age, String name) {
        this.age = age;
        this.name = name;
    }
	// 私有方法
    private Test(String name) {
        this.name = name;
    }
 
    public Test() {
    }
```

下面可以通过反射来获得私有内容：

```
// 新建一个类
Test test = new Test();
// 获得类内容
Class c4 = test.getClass();
// 新建构造方法，获得类的构造方法，可以有多个，所以是一个Constructor数组形式
Constructor[] constructors ;
constructors = c4.getDeclaredConstructors();

  	for (int i = 0; i < constructors.length; i++) {
		//getModifiers获得构造方法的类型
        System.out.print(Modifier.toString(constructors[i].getModifiers()) + "参数：");
		//getParameterTypes获得方法的入参类型
        Class[] parametertypes = constructors[i].getParameterTypes();
        for (int j = 0; j < parametertypes.length; j++) {
             	System.out.print(parametertypes[j].getName() + " ");
      	}
      System.out.println("");
  	}
```

上面使用的getDeclaredConstructors()是获得全部的构造方法

也可以直接获得特定构造方法，使用getDeclaredConstructor()，结尾没有s
```
	//要获取的是有两个参数int和String的构造方法，先设计好p，把p传入getDeclaredConstructor就好了
  Class[] p = {int.class,String.class};
  try {
       constructors = c4.getDeclaredConstructor(p);
       System.out.print(Modifier.toString(constructors.getModifiers()) + "参数:");
       Class[] parametertypes = constructors.getParameterTypes();
       for (int j = 0; j < parametertypes.length; j++) {
            System.out.print(parametertypes[j].getName() + " ");
          }
       } catch (NoSuchMethodException e) {
            e.printStackTrace();
     }
```

得到了构造方法之后，怎么调用构造一个新实例？（即我反射回来了我得用起来阿）

```
\\ 类内容
   public Test(int age, String name) {
        this.age = age;
        this.name = name;
        System.out.println("hello" + name + "i am" + age);
    }
 
 
    private Test(String name) {
        this.name = name;
        System.out.println("My Name is" +
                name);
    }
```

```
Test test = new Test();
Class c4 = test.getClass();

Class[] p = {int.class,String.class};
// 得到公共构造方法
constructors = c4.getDeclaredConstructor(p);
// 通过构造方法来新建实例
constructors.newInstance(24,"HuangLinqing");
// 得到并通过私有构造方法来新建实例
Class[] p = {String.class};
constructors = c4.getDeclaredConstructor(p);
// 注意要先setAccessible，不然打印不出来
constructors.setAccessible(true);
constructors.newInstance("HuangLinqing");
```

调用类的私有方法的方法：

假设有这么个刚发welcome，入参是String类型
```
  private void welcome(String tips){
        System.out.println(tips);
    }
```

可以这样调用他的私有方法：(使用invoke()执行，需要两个参数，类实例和方法参数，如果不能像new这样得到类实例的话，可以用上面的反射获得类信息，然后newInstance新建一个)
```
     Class[] p4 = {String.class};
     Method method = c4.getDeclaredMethod("welcome",p4);
     method.setAccessible(true);
     Object arg1s[] = {"test string"};
     method.invoke(test,arg1s);
```

获取私有字段并修改：

前提是没有办法得到类实例，不然的话直接test.set就好了。通过Filed类来做，getDeclaredField获得内部的私有字段，然后进行修改。
```
Field field = c4.getDeclaredField("name");
field.setAccessible(true);
field.set(o,"代码男人");
```

总结：

可以看到，基本上都是对一个类实例，反射得到内容，如果是public的，那么可以直接调用，如newInstance，invoke等，但如果是private的，就需要在调用前进行setAccessible(true)，然后才能使用。