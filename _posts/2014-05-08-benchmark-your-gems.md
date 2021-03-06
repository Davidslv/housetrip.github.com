---
layout: post
title: Benchmark RubyGem load times
published: true
author: David Silva
author_role: Associate Software Engineer
author_url: https://github.com/davidslv
author_avatar: http://www.gravatar.com/avatar/6aec36daee2fcb518971daa7f2e0f544.png
summary: |
  How can you benchmark the time that your gems take to load?
---

Maybe you are curious like me and you would like to check how long your gems take to load,
or maybe you have a considerable codebase which uses many gems and you are concerned
with the time it takes to load all of them.

At HouseTrip we have a rake task that allow us to benchmark the loading times of each gem on the web application,
which I would like to share with you.

The solution is fairly simple and makes use of the ruby builtin [Benchmark module](http://www.ruby-doc.org/stdlib-2.0/libdoc/benchmark/rdoc/Benchmark.html)
which allow us to see how long does it take to load each individual gem.

{% highlight ruby %}
# lib/tasks/benchmarks.rake
namespace "benchmarks" do

  desc "shows gem load times"
  task :gem_load_times do
    require 'rubygems'
    require 'bundler'
    require 'benchmark'
    require 'pry'

    ORIGINAL_ENV = ENV.clone
    $stderr      = StringIO.new

    reports = Benchmark.bmbm() do |x|
      x.report("Bundler.setup") { Bundler.setup }
      Bundler.setup.gems.each do |gem|

        ENV.replace(ORIGINAL_ENV)
        Gem.source_index.instance_variable_set(:@gems, {})
        x.report("#{gem.name} (#{gem.version})") do
          begin
            require gem.name.underscore
          rescue LoadError
            print "*"
          rescue
            print "**"
          end
        end
      end
    end

    puts "*: there was an error loading the gem"
    puts "**: there was an error after loading the gem"
  end

end
{% endhighlight %}
