在编程界有一句非常有名的格言：“对象是附带过程的数据，而闭包是附带数据的过程。”（Objects are data with post-attached methods, closures are functions with pre-attached data.）

要怎么来理解这句话,来看两段代码,分别使用类和闭包来实现一个累加器,每调用一次都累加的数字

下面是一段java代码
```
public class Incremental{
    private Integer count=0;

    public Integer incr(){
        return this.count++;
    }
}
Incremental myIncr=new Incremental();
myIncr.incr();
```

下面是一段go代码,是一个高阶函数
```

func genIncr()func()int{
  count:=0
  return func()int{
      result:=count
      count+=1
      return result
  }
}
myIncr:=genIncr()
```

两段代码在逻辑上基本是一样的,都是一个独立的函数/方法使用了一个无法直接被外部访问的变量,但是二者的侧重点是完全不一样的  
就如开头所说,二者的侧重点不同,我们使用类更多的是为了存储数据,就像是一个Person类,我们需要的不仅仅是方法,更多的是其中每个实例的名字,年龄,性别,而且非常容易拓展更多的方法.
而闭包,我们则是更关注过程,数据/状态只是次要.  

选用的时候,我们如果更需要记录一个**复杂的物体**,使用类;如果更需要一个轻量的**存在记忆的动作**,则选用闭包

当然,不光是意图上的区别,从其他的角度来考虑二者如何选用:  
- 性能上,类的在内存中的开销一般也比闭包更大,不过现在引擎上的升级二者的差距也没那么大了.  
- 实现方式上,二者各有千秋,有些语言不支持高阶函数,有些语言不支持类,虽然都有不同的替代方案,但是使用符合自己选用编程语言思想的方法是最好的.




