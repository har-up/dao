## 反射
- 虚拟机加载类时，会为每个类生成一个独一无二的Class对象,可以通过三种方式获取到类对象
  - 通过类的getClass()获取
    ```java
    class<?> c = new TextView().getClass()
    ```
  - 直接通过类名获取
    ```java
    Class<?> c = TextView.Class
    ```
  - 通过硬编码类名加载
    ```java
    try {
            Class<?> c = Class.forName("java.lang.String");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    
    //内部类
    try {
            Class<?> c   = Class.forName("android.view.View$OnClickListener");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    ```

- 通过反射验证项目是否使用了某个三方库
- 操作Field
  - getField和getDeclaredField
    getField和getDeclaredField都是获取类对象属性的方法，但二者有明显的区别
    - getDeclaredField配合setAccessible(true)可以获得本类中的任何Field
    - getDeclaredField无法获取父类中的任何field
    - getField只能获得本类或积累中的public修饰的Field

- 操作method
  ```java
   try {
            Class<?> c = Class.forName("java.lang.String");
            c.getMethod("setText",String.class).invoke(c,"hello");
        } catch (ClassNotFoundException | NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
  ```
- 考虑到反射相对来说是消耗性能多一点的，在适当的情形可以做缓存，比如ButterKnife在后面的版本就加入了反射的缓存，保证每个绑定类都放在缓存中，避免同一个绑定类
  多次反射。

- 动态代理
  - 实现InvocationHandler接口并创建自己的代理处理器
  - 为代理类传入classLoader和一组被代理接口
  - 通过反射机制获得动态代理类的构造函数
  - 利用构造函数创建动态代理类的实例

- CGLib
  上一节中的动态代理需要接口才能正常处理，如果我们想要代理的方法不是接口中定义的，而仅仅代理一个普通类的普通方法该如何实现呢，答案就是CGLib，它可以代理
  简单类，但不能代理final方法。
  > CGLib依赖ASM,即依赖于java字节码，所以Android无法使用，Android中类似的实现库有：zhangke3016/MethodInterceptProxy;linkedin/dexmaker;leo-ouyang/CGLib-for-Android

- 反射封装库 JOOR
- 反射的性能
  - set和get操作性能并不差
  - 用反射建立对象是比较耗时的
  - getMethod和getDeclaredField是最好是的方法
  > 6.0 ART环境提升了反射的性能
- 降低反射的性能消耗
  - 减少反射的使用
  - 利用APT代替反射，通过注解的形式在构建时通过注解找到对应的类生成对应的代码
- 反射与混淆，避免反射的对象被混淆，通过keep的方式保持反射的对象
