# java-notes
## 泛型程序设计
### 类型擦除
java泛型没有C++模板代码膨胀的问题，对于每一个泛型，都自动提供了相应的原始类型，擦除类型变量，将其替换为限定类型。
对于如下代码：
```
class Orig{
}

class Gnrc<T extends Orig>{
	public void testbrige(T t){
		System.out.println("Gnrc");
	}
}
```
并不会像C++那样，对于每一个类型都要展开，而是维护一个原始类型，它等价于：
```
class Gnrc{
	public void testbrige(Orig t){
		System.out.println("Gnrc");
	}
}
```
当没有extends的时候，一般原始类型是Object
这样，编译器会跟踪泛型实例，悄悄的为我们进行适当的类型转换
这种会导致一个问题，在上述代码的基础上，如果有如下代码：
```
class Deor extends Orig{
}

class Drv extends Gnrc<Deor>{
	public void testbrige(Deor s){
		System.out.println("covered");
	}
}
```
我们所期望的，当然是把Gnrc<Deor>当做一个新类使用，具有 public void testbrige(Deor s)这个方法，并且，在Drv中要覆盖它，然而Gnrc<Deor>并不存在public void testbrige(Deor s)这个方法，而是public void testbrige(Orig t)。。。
  <br />
事实上，编译器背后通过在Drv生成一个桥方法，它大致相当于(伪代码)：
```
  class Drv extends Gnrc<Deor>{
	  public void testbrige(Deor s){
		  System.out.println("covered");
	  }
    public void testbrige(Orig s){
		  testbrige( (Deor)s );
	  }
}
```
每当Drv实例调用public void testbrige(Deor s)，实际上是通过public void testbrige(Orig s)间接实现的。
    <br />
当然，如果非要钻个牛角尖，这种 泛型下，java确实不能实现C++泛型中的某些功能，例如，这时候就不能向Drv添加public void testbrige(Orig s)。。。
### 泛型约束和局限
#### 不能用基本类型实例化类型参数
因为基本类型不是Object类型的域，所以。。。
#### 运行时类型查询只能得到原始类型
#### 不能创建参数化类型的数组
先不考虑泛型数组，如果我们有如下代码：
```
String[] sarray=new String[10];
Object[] oarray=sarray;
//oarray[0]=new Orig();
```
注释掉的部分是不会被编译通过的，因为Orig和String不是同一个类型。然而，如果把String换成Pair<String>[], 把Orig换成Pair<Orig>，那么注释的部分可以通过编译，这是因为类型的擦除，使得sarray和oarray内的元素都是原始类型Pair。。。正是因为这样，编译器不允许下面语句通过：
```
Pair<String>[] sarray=new P    air<String>[10];
```
来避免把sarray赋给Object[]，而Object内的元素可以接受Pair的其它泛型。。。
