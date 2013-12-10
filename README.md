jsplugin
========

A modular javascript plugin boilerplate

When creating a plugin for jQuery use or as standalone js application or for use with the help of another library we have to address basic issues arising from the problem at hand:

- How it's gonna keep it's internal state during operation?
- Will it have memory between different calls?
- Will it handle elements and how?
- Will it store transient info for the changed elements?
- Will it handle events?
and so on.

But as we go deeper our questions are more on engineering design concepts:

- What patterns to use?
- Should it be expandable and how?
- Should be independable of libraries and how?

This plugin approach works well and can confronts difficulties like in the following scenario (see: http://stackoverflow.com/questions/7127600/which-jquery-plugin-design-pattern-should-i-use):

<blockquote cite="http://stackoverflow.com/questions/7127600/which-jquery-plugin-design-pattern-should-i-use"><i>we have nodes X, Y and Z that store an object reference into their <code>data</code> property like in the jQuery approach. Our objects operate on nodes.
<ul><li>Some of these objects store info in public <b>and/or</b> private accessible <b>internal member-objects</b>,</li>
<li>some classes of the member-objects might be connected with <b>inheritance</b>,</li>
<li>some of our member-objects might be private <b>and/or</b> public <b>singletons</b>.</li></ul></i></blockquote>

What would we do in such situation?
See the drawning:

<pre>
    classes : |  A           B         C
    ----------------------case 1----------
    members     |  |           |         |
      of        |  v           v         v
    an object   | var a=new A, b=new B,  c=new C
      at        |     B extends A
    node X :    |  a, b, c : private
    -----------------------case 2---------
    members     |  |           |         |
      of        |  v           v         v
    an object   | var aa=new A, bb=new B, cc=new C
      at        |     B extends A
    node Y :    |  aa, bb, cc : public
    -----------------------case 3--------
    Singletons: |  D, E
    -------------------------------------
    members     | var d= D.getInstance(),
      of        |     e= E.getInstance()
    an object   |
      at        | d: private,
    node Z :    | e : public
</pre>

This plugin boilerplate answers all these problems by combining two worlds:
- classical javascript but with a modular approach
- an external library like jQuery as a supplementary tool
- a jQuery plugin construction and
- the ability to use an <b>OO</b> class library to handle inheritance, object cloning etc. more easily!

See: http://jsfiddle.net/centurianii/s4J2H/1/

Issues
======
<ol>
<li>Still code duplicance?<br />
Even this demo is not cleared from ill behaviours: do you see a comment <code>MVC initial state</code> at file '<i>test-plugin.html</i>'?
It contains variable <code>var defaults = {...}</code> and you wonder: why should every node keep a seperate <code>defaults</code> variable? Think again! Yes, you can move it out of scope like <code>$$.A</code> as long as it doesn't contain node's specific data (here id does for demonstration purposes). Now you are getting it...
</li>
<li>Where is plugin chainability?<br />
I see, instead <code>$(element).data('myPlugin').get_a()</code> you want something like <code>$(element).myPlugin('get_a')</code>.
Think again, we decided to preserve jQuery chainability and not plugin, one is against the other. If you decide to break jQuery then this code:

<pre>
//extend jquery
   $.fn.myPlugin = function(options) {
      return this.each(function() {
         var plugin = myPluginFactory(this, options);
         $(this).data(plugin.pluginName, plugin);
      });
   };
</pre>

should be replaced by something like this one:

<pre>
//extend jquery
   $.fn.myPlugin = function(options) {
      var first;
      this.each(function(ndx) {
         var plugin = myPluginFactory(this, options);
         $(this).data(plugin.pluginName, plugin);
         if(ndx === 0) first = plugin;
      });
      return first;
   };
</pre>

and you have to add to your plugin a public method, say <code>method()</code> that acts as proxy for methods, i.e. accepting method name as argument but still, your chain would be: <code>$(element).myPlugin.method('get_a')</code> and only applicable to the first member of the node set.
</li>
<li>Why not using <code>$.myClass = {definition goes here...}</code>?<br />
Just change perspective: <code>$</code> is a function object, <code>$.myClass</code> adds in terms of OO design a <b>static</b> member, commonly accessible between different calls of jQuery or say it briefly you are moving <b>your</b> namespace under jQuery's <code>$</code>. It's a matter of preference here!
</li>
</ol>

Update
======
This approach tranformed the <a href="http://www.yuiblog.com/blog/2007/06/12/module-pattern/">module pattern</a> from a <b>self-invoked function</b> where the inner defined data behave as singletons to a <b>function</b> where every call reserves new memory resulting in duplicates of inner data one for every returned object.
Rephrasing the above leads us to a similar situation where functions are defined in the prototype versus functions that are defined inline of a constructor in terms of repetition. The first lacks repetition of data and the second one lacks unique, cross-object data and the winner is ... none.<br />
<blockquote><b>Move your static object members from the instance ones into external classes/singletons and then, reference them from a plugin like this one using a class factory that builds objects ready for storage.</b></blockquote>

Have fun!
