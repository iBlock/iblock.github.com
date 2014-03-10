---
layout: post
title: "测试博客"
date: 2014-03-10 10:47:23 +0800
comments: true
categories: 测试

---
这是一
个普通段落：

    <pre><code>NSString *str = @"hello";
	    NSLog(@"hello");
    </code></pre>

<!-- lang:c -->
    NSLog(@"NSString is red");
    int i = 0; 
	i = 1;  
	for (int i = 0; i < 100; i++) 
	{
		printf("");
	}

<!--more-->

*emphasize*   **strong**
_emphasize_   __strong__

An [example](http://www.baidu.com/ "Title")

An [example][id]. Then, anywhere
else in the doc, define the link:

  [id]: http://example.com/  "Title"

An email <example@example.com> link.

![alt text](/images/email.png "Title")

主题
========

标题
--------

# 标题1
## 标题2
### 标题3

1. Foo
2. Bar

*	A list item
	With multiple paragraphs
*	Bar

*   Abacus
    * answer
*   Bubbles
    1.  bunk
    2.  bupkis
        * BELITTLER
    3. burper
*   Cunning

> Email-style angle brackets
> are used for blockquotes.

> > And, they can be nested.

> #### Headers in blockquotes
> 
> * You can quote a list.
> * Etc.

`<code>` 
	
	NSString *str = @"hello world";
	NSLog(@"str = %@", str);
	
`` `this` ``.

`<code>` spans are delimited
by backticks.

You can include literal backticks
like `` `this` ``.

This is a normal paragraph.

    This is a preformatted
    code block.

{% codeblock Time to be Awesome - awesome.rb lang:objc %}
NSString *str = @"hello world";
NSLog(@"str = %@", str);
{% endcodeblock %}

``` ruby Discover if a number is prime http://www.noulakaz.net/weblog/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ Source Article
class Fixnum
  def prime?
    ('1' * self) !~ /^1?$|^(11+?)\1+$/
  end
end
```