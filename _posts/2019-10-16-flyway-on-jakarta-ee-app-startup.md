---
layout: post
title:  "Run Flyway migrations on Jakarta EE application startup"
date:   2019-10-16 14:31:20 +0200
categories: java
author: Tobias Erdle
---
While Spring has integrated support for database migration tools like Flyway, in a plain Jakarta EE
app everything needs to be bootstrapped by the developer. But thats easier than it seems, as the following code snippet shows:

{% highlight java %}
@Startup
@Singleton
public class FlywayMigrator {

    @Resource(lookup = "java:global/datasource-name)
    private DataSource dataSource;

    @PostConstruct
    public void doMigration() {
        final Flyway flyway = Flyway.configure().dataSource(dataSource).load();

        flyway.migrate();
    }
}
{% endhighlight %}

You simply need to implement a `javax.ejb.Startup` annotated bean (in the best case a `javax.ejb.Singleton`)
and configure and run the Flyway migration there.