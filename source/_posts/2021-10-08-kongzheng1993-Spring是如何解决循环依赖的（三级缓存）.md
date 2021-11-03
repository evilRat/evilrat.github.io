---
title: Spring是如何解决循环依赖的？
excerpt: 'Spring'
tags: [Spring]
categories: [Spring]
comments: true
date: 2021-10-08 16:30:10
---

## 什么是循环依赖

循环依赖分为三种，自身依赖于自身、互相循环依赖、多组循环依赖。但无论循环依赖的数量有多少，循环依赖的本质是一样的。就是你的完整创建依赖于我，而我的完整创建也依赖于你，但我们互相没法解耦，最终导致依赖创建失败。所以 Spring 提供了除了构造函数注入和原型注入外的，setter循环依赖注入解决方案。

```java

public class ABTest {

    public static void main(String[] args) {
        new ClazzA();
    }

}

class ClazzA {

    private ClazzB b = new ClazzB();

}

class ClazzB {

    private ClazzA a = new ClazzA();

}

```

这段代码就是循环依赖最初的模样，你中有我，我中有你，运行就报错`java.lang.StackOverflowError`。

#### 解决办法

```java

public class CircleTest {

    private final static Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

    public static void main(String[] args) throws Exception {
        System.out.println(getBean(B.class).getA());
        System.out.println(getBean(A.class).getB());
    }

    private static <T> T getBean(Class<T> beanClass) throws Exception {
        String beanName = beanClass.getSimpleName().toLowerCase();
        if (singletonObjects.containsKey(beanName)) {
            return (T) singletonObjects.get(beanName);
        }
        // 实例化对象入缓存
        Object obj = beanClass.newInstance();
        singletonObjects.put(beanName, obj);
        // 属性填充补全对象
        Field[] fields = obj.getClass().getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
            Class<?> fieldClass = field.getType();
            String fieldBeanName = fieldClass.getSimpleName().toLowerCase();
            field.set(obj, singletonObjects.containsKey(fieldBeanName) ? singletonObjects.get(fieldBeanName) : getBean(fieldClass));
            field.setAccessible(false);
        }
        return (T) obj;
    }

}

class A {

    private B b;

    // ...get/set
}

class B {
    private A a;

	// ...get/set
}
```

这里的解决方案就是将半成品对象，存放在缓存`singletonObjects`中，当B依赖A的时候能先使用半成品A来完成B的创建。

## Spring的三级缓存

```java

/**
 * Return the (raw) singleton object registered under the given name.
 * <p>Checks already instantiated singletons and also allows for an early
 * reference to a currently created singleton (resolving a circular reference).
 * @param beanName the name of the bean to look for
 * @param allowEarlyReference whether early references should be created or not
 * @return the registered singleton object, or {@code null} if none found
 */
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	// Quick check for existing instance without full singleton lock
	Object singletonObject = this.singletonObjects.get(beanName);
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		singletonObject = this.earlySingletonObjects.get(beanName);
		if (singletonObject == null && allowEarlyReference) {
			synchronized (this.singletonObjects) {
				// Consistent creation of early reference within full singleton lock
				singletonObject = this.singletonObjects.get(beanName);
				if (singletonObject == null) {
					singletonObject = this.earlySingletonObjects.get(beanName);
					if (singletonObject == null) {
						ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
						if (singletonFactory != null) {
							singletonObject = singletonFactory.getObject();
							this.earlySingletonObjects.put(beanName, singletonObject);
							this.singletonFactories.remove(beanName);
						}
					}
				}
			}
		}
	}
	return singletonObject;
}

```

- `singletonObjects.get(beanName)`，从`singletonObjects`获取实例，`singletonObjects`是成品bean
- `isSingletonCurrentlyInCreation`，判断beanName，`isSingletonCurrentlyInCreation` 对应的bean是否正在创建中
- `allowEarlyReference`，从`earlySingletonObjects`中获取提前曝光未成品的bean
- `singletonFactory.getObject()`，提前曝光bean实例，主要用于解决AOP循环依赖

### 只有一级缓存为什么不行？

一级缓存是存放bean的半成品。但是如果创建A的时候需要B，创建B的时候需要A，而A却还没有创建完成，会出现死循环。

### 二级缓存能解决问题吗？

一级缓存存放bean的成品，二级缓存存放bean的半成品。A在创建半成品对象后存放到缓存中，接下来补充A对象中依赖的B的属性。B继续创建，创建的半成品同样存放到缓存中，在补充对象的A属性的时候，可以从半成品缓存中获取A的半成品，那么现在B就是一个完成的对象了，而接下来递归操作后，A也是一个完整的对象了。

### 三级缓存解决了什么问题？

一级缓存是成品，二级缓存是半成品，三级缓存是工厂对象。二级缓存已经可以解决Spring的依赖了。三级缓存是为了解决Spring AOP的特性，AOP本身就是对方法的增强，是`ObjectFactory<?>`类型的lambda表达式，而Spring的原则又不希望将此类类型的Bean前置创建，所以要放到三级缓存中处理。其实整体处理过程类似，唯独是B在填充属性A时，先查询成品缓存、在查询半成品缓存，最后再看看有没有单例工程类在三级缓存中。最终获取到以后调用getObject方法返回代理引用或者原始引用。

