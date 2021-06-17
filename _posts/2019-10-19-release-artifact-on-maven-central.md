---
layout: post
title:  "Release artifact on Maven Central"
date:   2019-10-19 21:00:00 +0200
categories: java
author: Tobias Erdle
---

I'm working on a small Java lib on Github since a few weeks and thought it would be time to release it on Maven Central, so me or other people can use it without building it on their local machine. As it was a little bit hard for me to find all information necessary to perform the release, I decided to write down everything I did to get my lib into Maven Central.

If you want to follow the steps below, please make sure you have a working Java development environment (JDK, Maven) and a Github account. Ensure that `JAVA_HOME` is set properly.

## 1. Sign up on Sonatypes JIRA
The first step is to create an account in Sonatypes [JIRA](https://issues.sonatype.org){:target="_blank"}. There you need to create an issue in the second step.

## 2. Create issue for new project
In JIRA, you need to create an issue with type **New Project** and fill in the fields for your desired **groupId**, your project URL (your GitHub project link) and the Git URL (you need to use the **HTTPS** link). At the end, add your username and set *Already synced to Central* to **NO**. Create the issue.

**Small advice:** If you plan to release different artifacts under the same **groupId** (e.g. your domain), you can also ask for permissions on the "parent" *groupId*. So in my case, instead for asking only to get rights on *de.erdlet.jcrud*, I asked for access to *de.erdlet*.

For an example, please have a look into this [JIRA issue](https://issues.sonatype.org/browse/OSSRH-52453){:target="_blank"}.

### 2.1 Optional: Prove that you own your domain
If you want to publish artifacts on a *groupId* which is based on a custom domain, you need to prove Sonatype that you own (or better: have control over) the domain. Therefore set up a redirect to your GitHub Page (if you own one) or add a TXT DNS record referencing the issue.

## 3.0 Optional: Generate a GPG keypair and publish it
If you want to sign your artifacts and don't already have a GPG keypair you want to use for it, create one with `gpg --full-gen-key`. Afterwards, publish the public key to a keyserver, so other people can verify your builds.

## 4.0 Prepare and perform the release
The next steps describe how to prepare your project for the release and finally how to perform your release.

### 4.1 Ensure \<url\> and \<license\> is set
While performing my very first release on Sonatype, I forgot to declare these attributes and as a result, the artifact was not promoted to release repository.

### 4.2 Declare \<distributionManagement\>, \<issueManagement\> and \<scm\> attributes
While `<issueManagement>` and `<scm>` seem to be not that important for the release, the declaration of `<distributionManagement>` is more or less the most important for the Maven release. There is described where to push Snapshots and release artifacts. The `<distributionManagement>` section for Sonatype OSS looks like this:

```xml
  <distributionManagement>
    <snapshotRepository>
      <id>ossrh</id>
      <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
    <repository>
      <id>ossrh</id>
      <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/
      </url>
    </repository>
  </distributionManagement>
```

### 4.2 Declare plugins in pom.xml
After adding the information above, the next step is to add the necessary plugins for an artifact release. The order of the plugins in the `<build><plugin>` section is not important. At first, the **maven-deploy-plugin** is added:

```xml
      <plugin>
        <artifactId>maven-deploy-plugin</artifactId>
        <version>2.8.2</version>
        <executions>
          <execution>
            <id>default-deploy</id>
            <phase>deploy</phase>
            <goals>
              <goal>deploy</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
```

The second plugin is the **maven-release-plugin**:

```xml
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-release-plugin</artifactId>
        <version>2.5.3</version>
        <configuration>
          <localCheckout>true</localCheckout>
          <pushChanges>false</pushChanges>
          <mavenExecutorId>forked-path</mavenExecutorId>
        </configuration>
        <dependencies>
          <dependency>
            <groupId>org.apache.maven.scm</groupId>
            <artifactId>maven-scm-provider-gitexe</artifactId>
            <version>1.9.5</version>
          </dependency>
        </dependencies>
      </plugin>
```

The **maven-scm-provider-gitexe** plugin is used to generate the tags for the prepared release. For other SCMs are corresponding plugins available.

To get the sources and JavaDoc artifacts attached to the build, the **maven-source-plugin** needs to be set up:

```
     <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <version>3.1.0</version>
        <configuration>
          <encoding>UTF-8</encoding>
        </configuration>
        <executions>
          <execution>
            <id>attach-sources</id>
            <goals>
              <goal>jar</goal>
            </goals>
          </execution>
          <execution>
            <id>attach-javadoc</id>
            <goals>
              <goal>jar</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
```

And as a last plugin in the `<build>` section, the **nexus-staging-maven-plugin** can be added **optionally**. This plugin can be configured to perform the release from a staging repo automatically. If you want to do this process manually, skip this plugin.

```xml
      <plugin>
        <groupId>org.sonatype.plugins</groupId>
        <artifactId>nexus-staging-maven-plugin</artifactId>
        <version>1.6.7</version>
        <extensions>true</extensions>
        <configuration>
          <serverId>ossrh</serverId>
          <nexusUrl>https://oss.sonatype.org/</nexusUrl>
          <autoReleaseAfterClose>true</autoReleaseAfterClose>
        </configuration>
      </plugin>
```

After the `<build>` section is set up, a small profile for releases is added in the pom.xml. Inside, the plugin for signing the artifacts with the previously generated GPG key is configured. The whole section may look like this:

```xml
<profiles>
    <profile>
      <id>sign-artifacts-on-release</id>
      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>
      <build>
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>1.6</version>
            <executions>
              <execution>
                <id>sign-artifacts</id>
                <phase>verify</phase>
                <goals>
                  <goal>sign</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>
```
While preparing the release, you will be prompted for your GPG keys passphrase. If you want to release by a CI job, the passphrase can be set in the **settings.xml**. As I only want to release manually, I skipped this setting.

### 4.2 Add configuration to settings.xml
Before the release can be performed, there is a small change to do in the **settings.xml** (located in `~/.m2`).

One server attribute has to be added, which contains username and password for the Sonatype Nexus. These credentials are the same as the JIRA credentials created above.

```xml
   <servers>
        <server>
            <id>ossrh</id>
            <username>[your username]</username>
            <password>[your password]</password>
        </server>
    </servers>
```

Please pay attention to the **id**. It must be the same as the id declared in `<distributionManagement>`.

## 5. Prepare and release!
After these setup steps, the first release can be performed. Therefore the following commands have to be executed:

```bash
# Clean up your workspace
mvn clean

# Prepare the release. This call adds a Tag to git, sets the intermediate release
# version and increases the version in the pom afterwards.
mvn release:prepare

# Releases and deploys the plugin to Nexus.
# When the Nexus plugin is enabled, it also closes the staging repository and releases
# the artifact to the release repository.
mvn release:perform

```

### 5.1 Optional: Promote artifact manually
If you don't want to promote the artifact automatically to the release repository by the Nexus plugin, you need to log in to the [Sonatype OSS Nexus](https://oss.sonatype.org/){:target="_blank"} and do this step manually. The JIRA credentials have to be used for the Nexus login.

## 6. Verify repository and update the JIRA issue
After the `mvn release:perform` step is successful and the artifact promoted, check if it can be found in the Nexus. If yes, update the JIRA as claimed by the Sonatype admin, so the sync to Maven Central can be started and the artifact is going to be usable by everybody.

## Afterwards
After you released the lib, you just need to push the code changes (in this case it should only be the version uprade in the `pom.xml`) and the new Git tag to GitHub.
