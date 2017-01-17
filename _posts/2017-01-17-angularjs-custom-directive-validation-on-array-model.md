---
layout: post
title:  AngularJS Custom Directive Validation on Array Model
date:   2017-01-17 17:19:00 +0800
categories: angular
---

我们常常需要自定义一些输入控件，如选择用户控件，在angularjs中允许我们添加自定义directive，
我们用这种方式来实现自定义控件。

<iframe width="100%" height="450" src="https://jsfiddle.net/forhot2000/orx9fxd3/6/embedded/js,html,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

上面就可以简单的实现了我们需要的用户选择控件。

然而，我们经常需要判断选择的用户不允许为空，所以，我们来为它添加required validator，我们还是利用angular自带的
ngRequired directive来验证。

<iframe width="100%" height="450" src="https://jsfiddle.net/forhot2000/ny0e5w02/4/embedded/js,html,css,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
