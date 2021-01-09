---
layout: post
title:  "Use PyTorch machine learning framework in Java using Intellij and maven on MacOS"
date:   2020-12-09 01:35:00 +0800
category: backend
---
### Who is this post for ?

Software developers / machine learning engineers / data scients looking to get a pytoch model loaded into their java program. 

### Tools this post will be using : 

1. PyTorch
2. Oracle Java 1.8
3. Intellij
4. Maven
5. MacOS (10.13.6)

### Quick description about the tools :

Feel free to scroll to the next section if you are fairly familiar with the tools

1. PyTorch : 
   1. A machine learning framework that competes with the likes of Keras and Tensorflow
   2. Developed by Facebook reasearch
   3. Can also be used in place of numpy in GPU enabled environments.
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

If you like, source code for this entire post is available [here](https://github.com/siddharthmudgal/pytorch-java-maven-intellij-macos) and can be downloaded for **free**.

### Getting started : 

##### 	**Create a new Intellij maven project**



![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot1.jpeg?raw=true)



![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot2.jpeg?raw=true)

![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot3.jpeg?raw=true)

![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot4.jpeg?raw=true)



##### **Go ahead and add a new package and a main class inside it to run our sample code**



![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot5.jpeg?raw=true)

![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot6.jpeg?raw=true)

![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot7.jpeg?raw=true)

![Step one](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot8.jpeg?raw=true)



##### **Update the pom.xml in your project to look like this :** 

![pom.xml](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/shot9.jpeg?raw=true)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>playwithpytorch</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>


    <dependencies>
        <dependency>
            <groupId>ai.djl</groupId>
            <artifactId>api</artifactId>
            <version>0.8.0</version>
        </dependency>
        <dependency>
            <groupId>ai.djl</groupId>
            <artifactId>repository</artifactId>
            <version>0.4.1</version>
        </dependency>
        <dependency>
            <groupId>ai.djl.pytorch</groupId>
            <artifactId>pytorch-model-zoo</artifactId>
            <version>0.8.0</version>
        </dependency>
        <dependency>
            <groupId>ai.djl.pytorch</groupId>
            <artifactId>pytorch-native-cpu</artifactId>
            <classifier>osx-x86_64</classifier>
            <version>1.6.0</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
</project>
```



##### **Update Main.class to look like this**



```java
package com.playwithpytorch;

import ai.djl.Application;
import ai.djl.MalformedModelException;
import ai.djl.inference.Predictor;
import ai.djl.modality.cv.Image;
import ai.djl.modality.cv.ImageFactory;
import ai.djl.modality.cv.output.DetectedObjects;
import ai.djl.repository.zoo.Criteria;
import ai.djl.repository.zoo.ModelNotFoundException;
import ai.djl.repository.zoo.ModelZoo;
import ai.djl.repository.zoo.ZooModel;
import ai.djl.translate.TranslateException;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class Main {

    public static void main(String[] args) throws IOException, ModelNotFoundException, MalformedModelException {
        File inputFile = new File("/Users/siddharthmudgal/test/image.jpg");
        Image img = ImageFactory.getInstance().fromFile(inputFile.toPath());
        Criteria<Image, DetectedObjects> detectedObjectsCriteria =
                Criteria.builder()
                .optApplication(Application.CV.OBJECT_DETECTION)
                .setTypes(Image.class, DetectedObjects.class)
                .optFilter("backbone","resnet50")
                .build();

        try (ZooModel<Image, DetectedObjects> imageDetectedObjectsZooModel =
                     ModelZoo.loadModel(detectedObjectsCriteria)) {

            try (Predictor<Image, DetectedObjects> objectsPredictor= imageDetectedObjectsZooModel.newPredictor()) {

                DetectedObjects detectedObjects = objectsPredictor.predict(img);
                printDetectedObjectsToDisk(detectedObjects, img);

            } catch (TranslateException e) {
                e.printStackTrace();
            }

        }

    }

    public static void printDetectedObjectsToDisk(DetectedObjects detectedObjects , Image image) throws IOException {

        Path outDir = Paths.get("/Users/siddharthmudgal/test/output.jpeg");

        Image outputImage = image.duplicate(Image.Type.TYPE_INT_ARGB);
        outputImage.drawBoundingBoxes(detectedObjects);
        outputImage.save(Files.newOutputStream(outDir), "png");

    }

}
```



##### **Input image in my case looked like :** 

![input_image](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/image.jpg?raw=true)

##### **Output from the pytorch object detection looked like :** 

![input_image](https://github.com/thinknibbles/image_assets_for_thinknibbles/blob/main/pytorch_java_intellij_maven/output.png?raw=true)

​	*Although I am a little bummed that the label did not include Victoria Beckham, the model does produce decent object detection outputs.*



**I hope this post has helped you in some way.**

**Cheers!**