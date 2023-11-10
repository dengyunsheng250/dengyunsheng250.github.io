# Rust与Java泛型
在Rust中，泛型也会被删除。但两者的泛型仍然是不同的。

# Java泛型
在Java中，类的实例都是对象，并且每个对象都继承自`Object`。例如，如果你有一个Object数组，你可以在集合中存储任何内容。
```java
public class Container {
    public Object[] items;

    public Container(Object[] items) {
        this.items = items;
    }

}

class Dog {
    public Dog() {}
    public void bark() {}
}

class Main {
    public static void main(String[] args) {
        Dog d = new Dog();
        Container c = new Container(new Dog[] {d,d,d});
        c.items[0].bark(); compile fail
    }
}
```
main的最后一行不能通过编译，如下所示
```
error: cannot find symbol
        c.items[0].bark();
                  ^
  symbol:   method bark()
  location: class Object
```

如果将c显示转为`Dog`，它将编译。

因此，Java的泛型是不让你做某些事情。

![img](https://fasterthanli.me/static/img/rust-vs-java-generics/boxing.29acd1cd4d10ea59.png)

```java
void foobar<T>(Collection<T> coll) {
    // there is *no way* to know about T at runtime.
    // the compiler only wants to enforce that we use
    // coll correctly, but we cannot, for example, create
    // an instance of T. T's class is not stored anywhere
    // inside Collection.
    // At runtime, Collection<T> is Collection<Object>,
    // no matter what T is. It'll always take the same space,
    // and always use the same code.
}
```

如果在没有类型信息的情况下传递到其他地方，是不可能找到T是什么。

# Rust泛型
在Rust中，类型也会被擦除。
```rust
struct Container<T> {
    t: T,
}

fn foobar<T>(c: Container<T>) {
    // there is no way to know what T is at runtime.
    // we cannot match T. we cannot have different
    // codepaths for different T. The difference must
    // come from the outside.
}

fn main() {
    let a = Container { t: 42 };
}
```

这称之为，单态化，即多态化的代码生成带形式的代码。

同时，Rust也能得出变量的大小。
```rust
struct Container<T> {
    items: [T; 3],
}

fn main() {
    use std::mem::size_of_val;

    let cu8 = Container {
        items: [1u8, 2u8, 3u8],
    };
    println!("size of cu8 = {} bytes", size_of_val(&cu8));

    let cu32 = Container {
        items: [1u32, 2u32, 3u32],
    };
    println!("size of cu32 = {} bytes", size_of_val(&cu32));
}
```

输出
```
size of cu8 = 3 bytes
size of cu32 = 12 bytes
```

表现结果为
```rust
struct ContainerU8 {
    items: [u8; 3],
}

struct ContainerU32 {
    items: [u32; 3],
}

fn main() {
    let cu8 = ContainerU8 {
        items: [1, 2, 3],
    };

    let cu32 = ContainerU32 {
        items: [1, 2, 3],
    };

    // etc.
}
```

类似这样

![img](https://fasterthanli.me/static/img/rust-vs-java-generics/reify.5cd36c3171e25356.png)

单态化的一个特点是会增加二进制大小，但有利于内联。


# 总结

Java和Rust都做泛型擦除，但是....

Java中的泛型参数必须是对象

在Rust中，可以是任意大小类型

在Rust中，泛型类型和函数被统一，同时，优化器可能内联所有部分.
