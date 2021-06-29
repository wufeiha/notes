## Consumer

引用类调用不返回

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    ...
}
```

### 例子

```java
default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

## Function

### 定义

引用类调用返回任何类型

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    ...
}
```

### 例子

```java
Optional<String> nickOptional = getNickName("Justin");
out.println(nickOptional.map(String::toUpperCase));
```

## Predicate

### 定义

引用类调用返回boolean类型

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    ...
}
```

### 例子

```java

```



## Supplier

### 定义

无引用类调用返回

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

### 例子

```java
static void randomZero(Integer[] coins, Supplier<Integer> randomSupplier) {
    coins[randomSupplier.get()] = 0;
}
Integer[] coins = {10, 10, 10, 10, 10, 10, 10, 10, 10, 10};     
randomZero(coins, () -> (int) (Math.random() * 10));

```

