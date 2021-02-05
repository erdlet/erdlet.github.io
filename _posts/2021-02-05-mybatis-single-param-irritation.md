---
layout: post
title:  "MyBatis: There is no getter for property named... with single Parameter method"
date:   2021-02-05 09:00:00 +0200
categories: mybatis
author: Tobias Erdle
---

A few days ago, this exception was thrown in a MyBatis Mapper within a project: `There is no getter for property named 'foo' in 'class baz.bar.Foo'`.

A colleague and I had a look into the Mapper interface and XML, which had a similar content like the following snippet:

```java
void updateFoo(Foo foo);
```

```xml
<update id="updateFoo">
  UPDATE foo SET attr1 = #{foo.attr1} WHERE id = #{foo.id}
</update>
```

On the first sight, this seems to be correct. Afterwards, the we changed the Mapper to this:

```java
void updateFoo(Foo foo, Bar bar)
```

```xml
<update id="updateFoo">
  UPDATE foo SET attr1 = #{foo.attr1} WHERE id = #{foo.id} AND bar_id = #{bar.id}
</update>
```
To our surprise, this code worked. But why?

A few days later, I had a flash of thought: MyBatis is using a shortcut in the first example, because `foo` is the only parameter, 
so its attributes can be resolved directly using `#{attr1}`. So when we use only a single parameter, removing the parameter name seems to be mandatory instead
of being just a shortcut.
