---
layout: post
title:  "Create your first Hello World iOS App"
date:   2020-12-07 00:34:42 +0800
categories: [iOS, ObjectiveC]
tags: [iOS, ObjectiveC, MobileDevelopment]
---
This is a very straight forward and clean post to help fellow developers to successfully install and run their first iOS mobile app. 

# Prerequisites
- Xcode
- iPhone(optional)

# Agenda
To guide you to successfully create and run a very simple sample native iOS application on a device or simulator.

# Let's start
- Open the xcode in your machine. If you don't have xcode installed, please open `Appstore` app and search for `xcode` and click on install.
- Choose `File -> New -> Project` and choose `Single View App`
- Enter `Product Name`  `Organisation` of your choice. The product name will be the name of your app which thus can be altered later too.
- Click Next.
- Open `ViewController.m` and add line `#import<UIKit/UIKit.h>`
- Add below code snippet in the `viewDidLoad` to create a label with text message.


{% highlight ruby %}
UILabel *helloWorldLabel = [[UILabel alloc]initWithFrame:CGRectMake(91, 350, 300, 100)];
helloWorldLabel.text = @"Hello World";
[self.view addSubview:helloWorldLabel];
{% endhighlight %}

