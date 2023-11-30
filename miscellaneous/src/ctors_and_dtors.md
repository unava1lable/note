# 说明
现在有一个类`A`，和两个函数`init()`，`fini()`。
```cpp
class A {
public:
    A() {
        printf("hello from A\n");
    }
    ~A() {
        printf("bye from A\n");
    }
};

static A a;

__attribute__((destructor)) void fini() {
    printf("fini()\n");
}

__attribute__((constructor)) void init() {
    printf("init()\n");
}

int main(void) {
    return 0;
}
```
编译后运行：
```
init()
hello from A
bye from A
fini()
```
由于`a`已经析构了，所以我们没办法在`fini()`处理它。

# 方法(placement new)
```cpp
A *ap;

__attribute__((destructor)) void fini() {
    printf("fini()\n");
    ap->~A();
}

__attribute__((constructor)) void init() {
    printf("init()\n");
    char buffer_a[sizeof(A)];
    ap = new(buffer_a)A;
}
```
编译后运行：
```
init()
hello from A
fini()
bye from A
```