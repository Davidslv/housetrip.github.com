---
layout: post
title: mrd - MySQL in a RAMDisk for speedier tests
published: true
author: Julien
author_role: Developer
author_url: http://github.com/mezis
author_avatar: http://www.gravatar.com/avatar/88683f31bdf05a8071fb08327b3919cb.png
---

We try to live by the "red, green, refactor" mantra at HouseTrip. As a
consequence we have good test coverage, but our test suite is getting
larger. This is just one of the tricks that help us run our tests faster.

Many of our tests tend to use live ActiveRecord objects, which means the
database can be a speed bottleneck. Even with good SSDs, the heavy,
synchronous I/O performed when creating and modifying records is a grind.

A common idea is to make your test database's storage live in memory instead
of on-disk, and that's exactly what [mrd](https://github.com/mezis/mrd) sets
up for you.

Yes, [mrd](https://github.com/mezis/mrd) is pronounced like the french swear
word, (sorry if you find that in poor taste). The rationale here, is that
you might end up saying that word a lot (or the equivalent in your own
language) after manually setting up MySQL-in-a-RAMdisk a couple of times.

Fortunately, setting up `mrd` is trivial; just install like so:

{% highlight bash %}
  $ gem install mrd
  Successfully installed mrd-0.0.3
  1 gem installed
{% endhighlight %}

and run:

{% highlight bash %}
  $ mrd
  ==>
  Created Ramdisk at /dev/disk4
  Formatted Ramdisk at /dev/disk4
  Mounted Ramdisk at /Volumes/MySQLRAMDisk
  Starting MySQL server
  MySQL is now running.
  Configure you client to use the root user, no password, and the socket at '/Volumes/MySQLRAMDisk/mysql.sock'.
  Just close this terminal or press ^C when you no longer need it.
{% endhighlight %}

Then, if using Rails, point your `database.yml` to this new temporary SQL server:

{% highlight yaml %}
  test:
    ...
    socket: /Volumes/MySQLRAMDisk/mysql.sock
{% endhighlight %}

Don't forget to setup your test database:

{% highlight bash %}
  $ bundle exec rake db:create:all db:test:prepare
{% endhighlight %}

Voila! Slightly faster tests.

A couple of caveats to finish:

* mrd only supports MacOS, but you're very welcome to fork & extend for other platforms!
* This is a hastily hacked together script. I've been using it for six months without problems though.
