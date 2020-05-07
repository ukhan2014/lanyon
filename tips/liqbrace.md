---
layout: note
title: Liquid tags and curly braces in code 
---
<p class="message">
Liquid is confused by the curly braces, since they normally form Liquid tags.

To prevent an error, you'll need to wrap the JSON in Liquid raw tags, like this:
</p>
{% highlight js %}
{ % raw % }          // with no space bet { and %
{ {3478, udp}, ejabberd_stun, [] },
{ % endraw % }       // with no space bet { and %
{% endhighlight %}

