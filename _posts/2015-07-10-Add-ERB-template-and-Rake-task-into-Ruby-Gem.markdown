---
layout: post
title:  "Add ERB template and Rake task into Ruby Gem"
date:   2015-07-10 
categories: Ruby
---

When I was building ruby gem first time, I wanted to add rake task and ERB template into the gem. I had hard time finding the information at one place. 

To help others, I am jotting down the points which I used to add ERB template and tasks into the gem by using Railtie. Let me know if you found it useful.

**Step1. Generate Gem structure**
This creates the "foo" directory with the minimum gem structure.

`$ bundle gem foo`

foo
├── .gitignore
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── foo.gemspec
└── lib
  ├── foo
  │ └── version.rb
  └── foo.rb

**Step2. Update configuration:**
The file(foo.gemspec) provides metadata about the gem, like the name, description, author, license, and any gem dependencies required for it to work. Update it as per your requirements.
foo.gemspec
-----------------
{% highlight ruby %}

# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'foo/version'

Gem::Specification.new do |spec|
  spec.name          = "Foo"
  spec.version       = Foo::VERSION
  spec.authors       = ["Rohit Bhore"]
  spec.email         = ["rohitpbhore@gmail.com"]

  spec.summary       = %q{summary}
  spec.description   = %q{description}
  spec.homepage      = "homepage"
  spec.license       = "MIT"

  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]

  spec.add_development_dependency "bundler", "~> 1.8"
  spec.add_development_dependency "rake", "~> 10.0"
end
{% endhighlight %}

**Step3: Define Gem version**
lib/foo/version.rb
This file contains the version number. With every new release, it's necessary to update the version number to reflect the latest available version on RubyGems.org.

{% highlight ruby %}
module Foo
  VERSION = "0.0.1"
end
{% endhighlight %}

**Step4: Add any initializers**
lib/foo.rb
This is the file that is first loaded by default when your gem is required by Bundler.
It initially contains a reference to the version file. Mostly the code related to initializing the gem can be kept here.

{% highlight ruby %}
require "foo/version"

module Foo
 # Your code goes here...
end
{% endhighlight %}

**Step5: Build your Gem**
Now your gem base is ready. Go create your gem functionality. I can help you out by telling how to add tasks and html template in the gem.

How to add Templates?
Adding the ERB template at the location lib/foo/templates/template.html.erb
Now you can access the template via below code

{% highlight ruby %}
spec = Gem::Specification.find_by_name 'foo'
erb_file = "/#{spec.gem_dir}/lib/foo/templates/template.html.erb"
html_file = File.basename(erb_file, '.erb') 
erb_str = File.read(erb_file)

@var = 'var'
@far = 'far'

begin
  renderer = ERB.new(erb_str)
  result = renderer.result()
  File.open(html_file, 'w') do |f|
    f.write(result)
  end
rescue StandardError => e
  p e.message
  p e.backtrace
end
{% endhighlight %}

Here the instance variables @var and @far can be used directly into the template. The above code is used to generate a HTML file from the present ERB template into the Gem directory. Also you can use the template by adding controller action and their respective routes. In that case the instance variables will go into the controller's action.

How to add Rake Task?
Creating .rake file at the location lib/tasks/foo.rake

{% highlight ruby %}
require 'erb'

namespace :foo do
  desc "Run Foo"
  task :run do
    # Write ruby code here...
  end
end
{% endhighlight %}

lib/foo.rb
As this is the file that is first loaded by default, we should have to give reference of railtie here. Require railtie into Foo module.

{% highlight ruby %}
require "foo/version"
require 'foo/railtie' if defined?(Rails)

module Foo
end
{% endhighlight %}

Now to add task from the gem you need to create lib/foo/railtie.rb and load our .rake file.

{% highlight ruby %}
module Foo
  class Railtie < Rails::Railtie
     rake_tasks do
       spec = Gem::Specification.find_by_name 'foo'
       load "#{spec.gem_dir}/lib/tasks/foo.rake"
      end
   end
end
{% endhighlight %}

**Step6: Release Gem**
It's time to release now. build command builds a binary file with the gem name and version into the application's root folder.

`$ gem build foo.gemspec`

**Step7: Publish it**
We are able to create .gem file through above command.

`$ gem push foo-0.0.1.gem`

On the successful release, your gem will be accessible through RubyGems.org

**Finally: Use it**
Add Gem into application's Gemfile and bundle it.

`$ bundle`

Check for the availability of rake task

`$ rake -T`

You should have to see the task in the task list

`rake foo:run 	# Run Foo`

**References**
Loading rake tasks http://edgeapi.rubyonrails.org/classes/Rails/Railtie.html
Routes and controller for Gem http://www.smashingmagazine.com/2011/06/23/a-guide-to-starting-your-own-rails-engine-gem/
