# 使用BeanUtils.copyProperties遇到的java.lang.IllegalAccessException

今天在写项目的时候，在使用BeanUitls的时候遇到报红，看了一下发现是包用错了，用成了org.apache.commons.beanutils.BeanUtils

实际上应该用的是org.springframework.beans.BeanUtils



### 以下是IDEA的提示：

避免用Apache Beanutils进行属性的copy。 说明：Apache BeanUtils性能较差，可以使用其他方案比如Spring BeanUtils, Cglib BeanCopier。
    TestObject a = new TestObject();
    TestObject b = new TestObject();
    a.setX(b.getX());
    a.setY(b.getY());