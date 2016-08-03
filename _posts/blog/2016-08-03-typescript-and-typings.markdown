---
layout: post
title:  "Typescript and typings: A mystery to solve"
date:   2016-08-03
categories: blog
tags: [web,javascript,node,typescript]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

## What is Typescript?

When you code in javascript you don't know the datatype of variable used. You IDE doesn't know what's inside an object that's the reason it doesn't give you suggestion. For example many of the times you have to use `parseInt()` to convert a string/integer to a integer because you don't know what's actually in the variable.You can't create Class/Interface in javascript.Using javascript you can't write object oriented and modular code easily.That is where Typescript comes into role.
TypeScript is a typed superset of Javascript that compiles to plain javascript.Means you can write typed code(using classes/interfaces/datatypes and all) using typescript and later it can be compiled into javascript. Typescript gets compiled into javascript and then only it can be run, so basically at last you get pure javascript.You can also code nodejs using typescript.


__Example:__

<u>Typescript code</u>

{% highlight ts linenos %}

class Test{
	public x:number;
	constructor(x:number){
		this.x = x;

	}
	public printf():void{
		console.log(this.x);
	}
}

{% endhighlight %}

<u>Javascript code</u> 

(After compilation of Typescript code)

{% highlight javascript linenos %}

var Test = (function () {
    function Test(x) {
        this.x = x;
    }
    Test.prototype.printf = function () {
        console.log(this.x);
    };
    return Test;
}());

{% endhighlight %}


 When you start using typesript,at first you feel reluctant, but when you get used to it, it's pretty easy and can make your life simpler.For this post i am not covering the basics of typescript. You can read the basics from the [official documentation][typescript_docs] .

---------------------


## Namespaces and Modules in typescript:

Namespace <=> Internal Module

Module <=> External Module

In the starting versions of typescript `namespace` is called `internal module` and `module` is called `external module`.It's very confusing when you read some article/documentation/blog related to typescript so please keep that thing in mind.

__Module/External Module__

In TypeScript, just as in ECMAScript 2015, any file containing a top-level import or export is considered a module. For more info on module please check post [Javascript & Node : COMMONJS and AMD][javascript_modules] .

__Namespace/Internal Module__

Namespaces are using to basically wrap up the code in logical blogs.You can't `require` or `import` Namespaces in other modules/namespaces like you can import or require external modules. You can access Namespaces present in different files only through `reference path` .

<u>Example</u>

{% highlight ts linenos %}
namespace test{
	class A{
		public x:number;
	}
}
{% endhighlight %}

or 

{% highlight ts linenos %}
module test{
	class A{
		public x:number;
	}
}
{% endhighlight %}


You __can use both keywords namespace and module to create a namespace__ .As i mentioned above previously namespace was called internal module that's why there is a support of module keyword.

--------------------

## Sharing datatypes in Modules and Namespaces

Here i am describing how to share classes/interface basically datatypes between namespaces and modules which are written in same or different files.

__Case 1: Same Namespace in two different files__


file1.ts

{% highlight ts linenos %}
namespace x{
	export class Test{
		public a:number;
		constructor(a:number){
			this.a = a;
		}
	}
}

{% endhighlight %}

file2.ts

{% highlight ts linenos %}

namespace x{
    var test = new Test(2);
    console.log(test.a);
}

//compilation command
tsc file1.ts file2.ts
//tsc is typescript compiler.

{% endhighlight %}

__Case 2: Different Namespaces in two different files__

file1.ts

{% highlight ts linenos %}

module x{
	export class Test{
		public a:number;
		constructor(a:number){
			this.a = a;
		}
	}
	export var n = new Test(5);
}
{% endhighlight %}


file2.ts

{% highlight ts linenos %}

///<reference path="file1.ts"/>

var k = x.n;
console.log(k.a);

var  c = new x.Test(1);
console.log(c);

module y{
    var g = new x.Test(2);
    console.log(g);
}

{% endhighlight %}

__Case 3: External modules in Namespaces__

As you can't use `require` , `import` in namespaces so you can't use external modules types in internal modules.


__Case 4: Namespaces in External modules__

file1.ts:Internal module

{% highlight ts linenos %}

module check{
    export class Test{
        public a:number;
        constructor(){
            this.a = 5;
        }
    }
}

{% endhighlight %}

file2.ts:External module(commonjs)

{% highlight ts linenos %}

///<reference path="./file1.ts"/>
export var name = "shobhit";
var test = new check.Test();
console.log(test.a);

{% endhighlight %}

-----------------------

## Declaration Files:

Now the question arises is how to use existing javascript libraries like jquery, bootstrap etc. with typescript because with typescript you need typing to write code and later compile it.That's when we use declaration files .The extension of these file should be .d.ts .To describe the shape of libraries not written in TypeScript, we need to declare the API that the javascript library exposes. We call declarations that don’t define an implementation `ambient`.These files are just for typing purposes,code in these files does't get compiled nor it gets loaded into browser.

Some interesting points related to decleration (.d.ts) files:

1. Have to use `declare module module_name { ... }` not module module_name{ ... }.Because you are not writing a namespace but actually writing a ambient module which is just used for declaration purposes. This type of module which starts from declare is called `ambient module`.

2. Can't initialize anything in this file.As this file doesn't get compiled,you can't not use the initialized data in some other file.In the same way you can't create constructor in class in ambient modules.It is recommended to use interfaces in these module as you can't leverage the class functionality in these modules.

3. You can write external modules too in this file but with point 2 restriction.

<u>Examples:</u>

file3.d.ts

{% highlight ts linenos %}

declare module x{
    export interface Test{
        a:number;
    }
}
{% endhighlight %}

file2.ts

{% highlight ts linenos %}

///<reference path="file3.d.ts"/>
var text:x.Test = {a:10};
console.log(text.a);

{% endhighlight %}

Check, here we are using the typings of file3.d.ts in file2.ts using `reference path` . __Reference path can only be given in case of internal modules and ambient modules.__ If .d.ts file contains external module then you have to use `import {Type} from "filepath"` to use in other files.Check the example:


file3.d.ts

{% highlight ts linenos %}
export interface Test{
    x:number;
}
{% endhighlight %}


file2.ts

{% highlight ts linenos %}
import {Test} from "./file3.d.ts";
export class C {
    public element : Test;
}
{% endhighlight %}


[typescript_docs]: https://www.typescriptlang.org/docs/tutorial.html
[javascript_modules]: {{ site.url }}/blog/commonjs-and-amd/
