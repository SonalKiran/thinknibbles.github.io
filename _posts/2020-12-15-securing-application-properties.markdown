---
layout: post
title:  "Securing application properties with jasypt spring boot maven java intellij macos"
date:   2020-12-15 01:35:00 +0800
categories: [Backend-development, Java, Jasypt, Encryption, Spring-boot]
tags: [Java, Backend-development, Intellij, Machine learning, pytorch, macOS, Jaspyt, Encryption, Spring-boot]
---
### Who is this post for ?

Backend software developers working towards making their code production ready or anyone looking to conceal property source files.

### Tools this post will be using : 

1. Jasypt 3.0.3
2. Spring boot framework 2.3.4.RELEASE
3. Oracle Java 1.8
4. Intellij
5. Maven
6. MacOS (10.13.6)

### Quick description about the tools :

Feel free to scroll to the next section if you are fairly familiar with the tools

1. Jasypt : 
   1. Open source project that provides encryption support for property sources in spring boot applications
2. Spring boot framework
   1. An open source java based framework that helps create production ready microservices with a lot of amazing features like dependency management, health checks, metrics, embedded tomcat and external configurations
3. Oracle Java 1.8 :
   1. A very popular programming language
   2. OpenJDK is oracle java's open source implementation
4. Intellij :
   1. A popular IDE used for Java
   2. Pretty power when it comes to debugging and addon integration
   3. Comes in 2 versions : Community and Ultimate
5. Maven
   1. Build automation tool
   2. Makes dependency management pretty convinient
6. MacOS
   1. A popular operating system



### Source Code 

If you like, source code for this entire post is available [here](https://github.com/siddharthmudgal/jasypt-springboot-maven-java-intellij-macos) and can be downloaded for **free**.

### Getting started : 

##### 	**Create a new Intellij maven project**



![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot1.jpeg?raw=true)



![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot2.jpeg?raw=true)

![step three](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot3.jpeg?raw=true)



##### **Go ahead, and throw in a package name and a Main class**

![step four](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot4.jpeg?raw=true)

![step five](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot5.jpeg?raw=true)

![step six](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot6.jpeg?raw=true)

![step seven](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot7.jpeg?raw=true)



##### **Let's also add an application.properties file to store some properties**

![step eight](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot8.jpeg?raw=true)

![step nine](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot9.jpeg?raw=true)



##### **Add a new property to application.properties, it should look like this :** 

```properties
sample.property=now_you_see_this_property
```



##### **Your project structure should look like this :** 

![step ten](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot10.jpeg?raw=true)



##### **Initialise your pom.xml with the following values**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.thinknibbles</groupId>
    <artifactId>jasypt-springboot-maven-java-intellij-macos</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <parent>
        <!-- marking parent as spring boot starter -->
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath></relativePath>
    </parent>

    <dependencies>
        <!-- jasypt maven depdency -->
        <dependency>
            <groupId>com.github.ulisesbocchio</groupId>
            <artifactId>jasypt-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <!-- required to encrypt sources files or single values via maven -->
                    <groupId>com.github.ulisesbocchio</groupId>
                    <artifactId>jasypt-maven-plugin</artifactId>
                    <version>3.0.3</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>

</project>
```



##### **Next, go ahead and configure your Main class to act as the SpringBootApplication class :** 

```java
package com.thinknibbles;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class Main extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }

}
```



##### **Next, add a new configuration class to our project, which will allows us to load property sources**

![step eleven](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot11.jpeg?raw=true)

![step twelve](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot12.jpeg?raw=true)



##### **Initialise the configuration class with the following code :**

```java
package com.thinknibbles;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;

@Configuration
@PropertySource(value = "application.properties")
public class Configuration_environmentproperties {

    Environment environment;

    private final String TAG = " (Configuration_environmentproperties) ";
    private String sample_property;
    private final String sample_property_key = "sample.property";

    @Autowired
    public Configuration_environmentproperties(Environment environment) {
        this.environment = environment;
        this.sample_property = this.environment.getProperty(sample_property_key);
        System.out.println(TAG + "sample_property value = " + sample_property);
    }
    public String getSample_property(){
        return this.sample_property;
    }

}
```



##### **At this stage, we've successfully setup a spring boot project with properties being loaded from application.properties. The project structure should now look like this :** 

![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot13_1.jpeg?raw=true)



##### **Go ahead and run the program once to verify if the properties are begin correctly loaded**

![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot13.jpeg?raw=true)

![step twelve](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot14.jpeg?raw=true)

![step twelve](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot15.jpeg?raw=true)



##### **Now go ahead and click on run :**

![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot16_1.jpeg?raw=true)



##### **Your application logs should contain the following line :** 

```verilog
(Configuration_environmentproperties) sample_property value = now_you_see_this_property
```



##### **Screenshot from my sample application**

![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot17.jpeg?raw=true)



##### **If you see the log printed, then you too have successfully estabilished an end to end pipeline. Congratulations! Next, let's encrypt the exisiting properties in the application.properties file**



##### **Add DEC prefix to the existing value of sample.property with parenthesis, it would look something like this :** 

```properties
sample.property=DEC(now_you_see_this_property)
```

##### **This would mark this property ready for encryption**



##### **Now, go ahead and run the following command from your terminal ( from the root directory of your project ) :**

```bash
mvn jasypt:encrypt -Djasypt.encryptor.password="randompasswordhash"
```

##### **This starts execution of jasypt plugin that we configured in our pom.xml and searches for all DEC() marked properties and proceeds to encrypt them. Once the execution of the above command finishes, you should see all the properties with DEC updated to ENC().** 



##### **Post execution of the command our sample.property would now look like this in the application.properties files :**

```properties
sample.property=ENC(ViNAh+QfUYzK6q1QUSzOdf8ZwLN8jlCHLxHrXuNVuDH+9swLQ/ycNfbyHT24oA9Y9XenGEdL8m7240etXffLxg==)
```



##### **Let's try running our spring application now to see if it can load the encrypted property. Following huge a** error should pop up in the terminal :**

```verilog
Error creating bean with name 'main': Unsatisfied dependency expressed through field 'configuration_environmentproperties'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'configuration_environmentproperties' defined in file [/Users/siddharthmudgal/webportfolio/jasypt-springboot-maven-java-intellij-macos/target/classes/com/thinknibbles/Configuration_environmentproperties.class]: Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.thinknibbles.Configuration_environmentproperties$$EnhancerBySpringCGLIB$$fc5e894f]: Constructor threw exception; nested exception is java.lang.IllegalStateException: either 'jasypt.encryptor.password' or one of ['jasypt.encryptor.private-key-string', 'jasypt.encryptor.private-key-location'] must be provided for Password-based or Asymmetric encryption
```



##### **If you see this error, then you're on the right track buddy. Here, the spring framework is attempting to load an encrypted property, but it doesn't have the 'password' that was used to encrypt it. Without further ado, let's go ahead and populate <u>jasypt.encryptor.password</u> in the system environment to let spring framework know the password ( in our case -> 'randompasswordhash' )**



![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot18.jpeg?raw=true)

![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot19.jpeg?raw=true)

![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot20.jpeg?raw=true)



##### **Now, let's try to run our program again. You should once again see the following line in the logs :** 

```verilog
 (Configuration_environmentproperties) sample_property value = now_you_see_this_property
```

![step](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/jasypt_springboot_java/shot21.jpeg?raw=true)

##### **As you can see, that the original property value that we had encrypted can now be read by our application.**





#### **Voila! now you've been enabled with a new weapon of mass concealement. I hope this post has helped you in some way.**

#### **Cheers!**