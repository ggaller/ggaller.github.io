---
layout: post
title:  "Distribution algorithms. The beginning"
date:   2020-05-11 15:09:28 +0300
categories: algorithms distribution hashing
---

Modulo hashing

{% highlight csharp %}
public T FindNode(IUniqueObject obj)
{
    var hash = obj.GetHashCode();
    return _nodes[Math.Abs(hash % _nodes.Count)];
}
{% endhighlight %}