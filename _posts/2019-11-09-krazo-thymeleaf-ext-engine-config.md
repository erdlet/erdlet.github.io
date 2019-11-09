---
layout: post
title:  "Krazo Thymeleaf extension: How to override default TemplateEngine producer"
date:   2019-11-09 14:00:00 +0200
categories: java
author: Tobias Erdle
---

While I was working on a small MVC application during the last weeks, I struggled a little
bit when trying to override the default produced `TemplateEngine` of Krazo's Thymeleaf extension. This post will
explain, how to override the default to be able to configure Thymeleaf add-ons like the layout dialect.

In my application, I use this class to override the `DefaultTemplateEngineProducer` of Krazo's Thymeleaf extension:

```java

import javax.enterprise.inject.Produces;
import javax.enterprise.inject.Specializes;

import org.eclipse.krazo.ext.thymeleaf.DefaultTemplateEngineProducer;
import org.thymeleaf.TemplateEngine;

import nz.net.ultraq.thymeleaf.LayoutDialect;

public class CustomThymeleafEngineProducer extends DefaultTemplateEngineProducer {

    @Override
    @Produces
    @Specializes
    public TemplateEngine getTemplateEngine() {
        
        final TemplateEngine templateEngine = super.getTemplateEngine();
        templateEngine.addDialect(new LayoutDialect());

        return templateEngine;
    }
}
```

There is nothing really special about it on the first look, as it simply overrides `DefaultTemplateEngineProducer` and `getTemplateEngine`.
But on a second look, you might see the `javax.enterprise.inject.Specializes` annotation on the producer method. This annotation leads to 
a deactivation of the default `TemplateEngine` in the CDI container and a inject of our custom `TemplateEngine` in all consumers.
