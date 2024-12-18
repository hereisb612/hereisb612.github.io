# Lambda
## What is Lambda

```java
interface Lamda {  
    int calculate(int a, int b);  
}  
  
public class WhatIsLambda {  
    public static void main(String[] args) {  
  
        Lamda lamda = (a, b) -> a + b;  
        System.out.println(lamda.calculate(1, 2));  
    }  
}
```

在本例中，`(a, b) -> a + b` 是函数本身，传入参数 a 和 b，return a + b。其参数和返回值属性可以通过接口中的方法来确定。

该表达式用多态接受了一个 interface Lamda 类型的 **函数对象**，该函数对象可以作为参数传递，这是 **函数化对象** 的意义。

通过接口中的方法名，即可以调用该方法。