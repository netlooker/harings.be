To improve php performance, it's good to start profiling the application you are working on, as there might be some bad code that heavily decreases performance.

One tool to do this with, is Xhprof. Xhprof is a tool that helps you detect possible code issues, it accomplishes this by showing you what function get called most, how long it takes, memory and cpu usage per call etc etc.

I'm going to show you how you can upgrade your environment to enable Xhprof in an easy way, (One time setup).

As I am working with OS X, I'll be using homebrew to install xhprof, for additional information on how to install xhprof you can look <a href="https://pecl.php.net/package/xhprof">here</a>.

### Install xhprof
Assuming that you already have php (5.6) configured, we are now installing xhprof with the following command.

```
brew install php56-xhprof
```

Now that we have xhprof installed, we can go on and install a tool to look at the data.

### Install Xhgui
[Xhgui](https://github.com/preinheimer/xhprof "xhgui") is a tool that captures the profiling data to a database. It's easy to install, just follow the instructions you can find on the github page.

[Installation instructions](https://github.com/preinheimer/xhprof#installation "Installation instructions")

But skip the "auto_prepend_file" step.

A tip in this section:
* set **$_xhprof['servername']** to something short.


### Chrome extension
To enable per site profiling, we need a Cookie.

In chrome this can be easily done by using the [Xhprof helper](https://chrome.google.com/webstore/detail/xhprof-helper/adnlhmmjijeflmbmlpmhilkicpnodphi "Xhprof helper").

Please see your browser of preference for a similar tool.

### Glue!
All tools in place, we will glue them together.

In your apache virtualhost config, we'll add a new section. Make sure it looks a bit like the example below:

```
<Virtualhost *:80>
  VirtualDocumentRoot "/Users/<USER>/Sites/localhost/%1/public"
  ServerName sites.loc
  ServerAlias *.loc
  UseCanonicalName Off
  <IfModule mod_php5.c>
     php_value auto_prepend_file "/Users/<USER>/Sites/localhost/xhprof/external/header.php"
  </IfModule>
</Virtualhost>
```

As you can see, I have Xhgui installed in **/Users/<USER>/Sites/localhost/xhprof/external/header.php**

What does it do?

```
php_value auto_prepend_file "/Users/<USER>/Sites/localhost/xhprof/external/header.php"
```

This line automatically prepends the xhprof header file to all pages. In it's current state, this will start profiling every page (which is not what we want),  so lets go ahead and change this file a bit.

So, open up /Users/**USER**/Sites/localhost/xhprof/external/header.php with your favorite editor.

And wrap the full file in an IF statement.

```
<?php
if (isset($_COOKIE["_profile"])) {
  // Original code.
}
```

The same in /Users/**USER**/Sites/localhost/xhprof/external/footer.php.

```
<?php
if (isset($_COOKIE["_profile"])) {
  // Original code.
}
```

The changes we made above, will make the toggle button, that we installed into chrome, will trigger the profiler to start.

![Xhprof button](http://harings.be/sites/default/files/2016-02/Schermafbeelding%202016-02-22%20om%2012.55.46_7.png)

If it is not active, we just use the pages as normal without any performance issues.
