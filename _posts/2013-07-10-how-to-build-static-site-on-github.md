---
layout: post
category : intro
tagline: "Supporting tagline"
tags : [jekyll, github]
---
{% include JB/setup %}

**Jekyll** is a ruby tool. You can build a static website on github like this One with it.


The back-end engine of the feature ***Github Pages*** is jekyll. If your github project contains the right jekyll configuration, it can be recognized by Github. Thus you can build a blog website on Github for free.
 

ruby has a install management tool named gem, in ubuntu you can install gem by executing

    sudo apt-get install rubygems


After that, install jekyll by gem

    sudo gem install Jekyll

 
During the installation, i have encounterred a few problems.

**Problem 1**

    hailin@hailin-VirtualBox:~/win7/MyProjects $sudo gem install jekyll
    extconf.rb:1:in `require': no such file toload -- mkmf (LoadError)



In order to fix this problem, you have to install the follow dependency.

    $ sudo apt-get install build-essential libopenssl-rubyruby1.8-dev

 
The answer is found on [http://ruby.about.com/od/faqs/qt/Extconf-Rb-1-In-Require-No-Such-File-To-Load-Mkmf-Loaderror.htm](http://ruby.about.com/od/faqs/qt/Extconf-Rb-1-In-Require-No-Such-File-To-Load-Mkmf-Loaderror.htm)

 
**Problem 2**

    hailin@hailin-VirtualBox:~/win7/MyProjects $sudo gem install jekyll
    Successfully installed jekyll-1.0.3
    1 gem installed
    Installing ri documentation forjekyll-1.0.3...

    ERROR: While generating documentation for jekyll-1.0.3

    ... MESSAGE:   Unhandled special: Special: type=17,text="<!-- more -->"
    ... RDOC args: --ri --op/var/lib/gems/1.8/doc/jekyll-1.0.3/ri --charset=UTF-8 --quiet libREADME.textile LICENSE --title jekyll-1.0.3 Documentation
    (continuing with the rest of theinstallation)

    Installing RDoc documentation forjekyll-1.0.3...

 
It seems jekyll installed successfully, but ri documentation failed.

In [http://stackoverflow.com/questions/1381725/how-to-make-no-ri-no-rdoc-the-default-for-gem-install](http://stackoverflow.com/questions/1381725/how-to-make-no-ri-no-rdoc-the-default-for-gem-install), it said you can skip installing the document by creating a file  named **~/.gemrc**, with the content is:

    gem: --no-ri --no-rdoc
 



And then i tried again, another problem happened.

**Problem 3**

    hailin@hailin-VirtualBox:~/win7/MyProjects $sudo gem install jekyll
    Successfully installed jekyll-1.0.3
    1 gem installed

    Installing ri documentation forjekyll-1.0.3...

    ERROR: While executing gem ... (NoMethodError)
    undefinedmethod `map' for Gem::Specification:Class

 
It seems the definition for method 'map' cannot be found.

As jekyll is installed with sudo, the file is installed in the directory **/var/lib/gems/1.8**, but this directory is not on the PATH env by default.

    hailin@hailin-VirtualBox:~/win7/MyProjects$ls /var/lib/gems/1.8/gems/jekyll-1.0.3/
    bin CONTRIBUTING.md  cucumber.yml  features Gemfile  History.markdown  jekyll.gemspec  lib LICENSE  Rakefile  README.textile  script site  test

So, i add this directory to the env PATH. (add the following line in /etc/profile).

    export PATH=$PATH:/var/lib/gems/1.8/bin #Add RVM to PATH for scripting

 
At last, the jekyll works! 

**Create a blog**

creat a site by **Jekyll new myblog**. It will generate default files/folders，mainly includes

    _config.yml : jekyll main configuration
    _layouts : layout template
    _post : blog post directory



you can start the website by **Jekyll serve**, and then visit the page on 127.0.0.1:4000.


In order to make the blog working on github, the command should be **jekyll new YOU-GITHUB-ID.github.com**

    hailin@hailin-VirtualBox:~/win7/MyProjects$jekyll new hailinzeng.github.com
    New jekyll site installed in/home/hailin/win7/MyProjects/hailinzeng.github.com


Create a project named YOU-GITHUB-ID.github.com on github, and commit the above folder. After that, you will able to see it on github on http://YOU-GITHUB-ID.github.com.


**Add new blog post**

The blog post are put in the directory "_post", the filename should be year-month-day-title.md. You can edit the md(MarkDown) file by any text editor. I propose **MarkDownPad**.


In the front end of each blog post, there are some meta data, which is called yaml header. The are enclosed by two lines of "---", which means a beginning and ending of the meta data. Between then, you can set one meta data in each line.


For more configuration of jekyll, you can visit [http://jekyllrb.com/docs/structure/](http://jekyllrb.com/docs/structure/)


ps: If you want to have you website better looking, but lazy to do it yourself(like me). you can fork others projects directly, like the jekyllbootstrap project [http://jekyllbootstrap.com/](http://jekyllbootstrap.com/)
