---
layout: post
title:  "Guide to start using bytedeco Opencv"
date:   2021-03-28 01:35:00 +0800
categories: [Backend, Java, OpenCV, ImageProcessing, bytedeco, computerVision]
tags: [Java, BackendDevelopment, Intellij, Computer Vision, OpenCV, macOS, bytedeco, SpringBoot, maven]
---
### Who is this post for ?

Backend Java developers looking to start using bytedeco OpenCV libraries.

### Tools this post will be using : 

1. JavaCV ( bytedeco ) - 1.5.5
2. Oracle Java 1.8
3. Intellij
4. Maven
5. MacOS (10.13.6)

### Quick description about the tools :

Feel free to scroll to the next section if you are fairly familiar with the tools

1. JavaCV (bytedeco) : 
   1. Open source project that provides wrapper over native OpenCV libs ( packaged along with native OpenCV libraries for all platforms )
2. Oracle Java 1.8 :
   1. A very popular programming language
   2. OpenJDK is oracle java's open source implementation
3. Intellij :
   1. A popular IDE used for Java
   2. Pretty power when it comes to debugging and addon integration
   3. Comes in 2 versions : Community and Ultimate
4. Maven
   1. Build automation tool
   2. Makes dependency management pretty convinient
5. MacOS
   1. A popular operating system



### Source Code 

If you like, source code for this entire post is available [here](https://github.com/siddharthmudgal/bytedeco-opencv-sample) and can be downloaded for **free**.

### Getting started : 

##### 	**Create a new Intellij maven project**



![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot1.jpeg?raw=true)



![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot2.jpeg?raw=true)

![step three](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/bytedeco_opencv_java_intellij_maven/shot3.png?raw=true)



##### **Go ahead, and throw in a package name and a Main class**

![step four](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/bytedeco_opencv_java_intellij_maven/shot4.png?raw=true)

##### **Here is what your pom should look like currently**

![step five](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/bytedeco_opencv_java_intellij_maven/shot5.png?raw=true)


##### **Initialise your pom.xml with the following values**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>opencvDistributableByteDeco</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- dependency required to start using bytedeco -->
    <dependencies>
        <dependency>
            <groupId>org.bytedeco</groupId>
            <artifactId>javacv-platform</artifactId>
            <version>1.5.5</version>
        </dependency>
    </dependencies>

</project>
```

##### **Your pom.xml should be looking something like this**

![step six](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/bytedeco_opencv_java_intellij_maven/shot6.png?raw=true)


##### **Hit maven reload and external dependencies should have been downloaded in your external libs folder**

![step seven](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/bytedeco_opencv_java_intellij_maven/shot7.png?raw=true)


##### **Next, go ahead and configure your Main class :** 

```java
package org.example;

/**
 * make sure all imports are from bytedeco package name and not org.opencv
 * org.opencv package functionalities will not work with bytedeco
 */

import org.bytedeco.javacpp.Loader;
import org.bytedeco.opencv.opencv_core.Mat;
import org.bytedeco.opencv.opencv_java;

import static org.bytedeco.opencv.global.opencv_imgcodecs.imread;

/**
 * main class
 */
public class Main {

    public static void main(String[] args) {

        // this loads bytedeco native libraries into the memory. First Step!
        Loader.load(opencv_java.class);

        //notice how imread is different from the original OpenCV jar wrapper provided by OpenCV org
        Mat mat = imread(Main.class.getClassLoader().getResource("sample.jpeg").getPath());

        if (mat == null)
            System.err.println("Unable to read image");

        System.out.println("Image dimensions -> " + mat.cols() + " x " + mat.rows());
        mat.release();

    }

}

```
##### **Run your main class and your output should look like what is being displayed in the console**

![step eight](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/bytedeco_opencv_java_intellij_maven/shot8.png?raw=true)


#### **Voila! now you've been enabled with a new weapon of mass distribution and will definitely help you in most production scenarios. I hope this post has helped you in some way.**

#### **Cheers!**