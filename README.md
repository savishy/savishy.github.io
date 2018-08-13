# Github Page for Vish (savishy.github.io)

This is the source for the Github-hosted website http://savishy.github.io.

This readme contains instructions on building/running this site, as well as references.

## How to run (locally, for development)

* The site runs on Jekyll.
* You need Ruby in order to run Jekyll
* It's recommended to install Brightbox Ruby 2.5 (as well as the dev package).
* Install Bundler.
```
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt install ruby2.5 ruby2.5-dev
sudo gem install bundler
```

* Clone this repository.
* Execute the Bundler commands.

```
git clone http://github.com/savishy/savishy.github.io
cd savishy.github.io/
bundle install
bundle exec jekyll serve
```

Now browse to `http://localhost:4000`.

## If your build fails with `zlib is missing; necessary for building libxml2`

```
zlib is missing; necessary for building libxml2

An error occurred while installing nokogiri (1.8.1), and Bundler cannot continue.
Make sure that `gem install nokogiri -v '1.8.1' --source 'https://rubygems.org/'` succeeds before bundling.
```

**Solution:**

https://github.com/flapjack/omnibus-flapjack/issues/72

`sudo apt install zlib1g-dev`

## If your `bundle exec jekyll serve` command fails with `libcurl.so` errors

```
vishy@freeman:/mnt/c/Users/savis/work/savishy.github.io$ /usr/local/bin/bundle exec jekyll serve
Configuration file: /mnt/c/Users/savis/work/savishy.github.io/_config.yml
  Dependency Error: Yikes! It looks like you don't have jekyll-remote-theme or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'Could not open library 'libcurl': libcurl: cannot open shared object file: No such file or directory. Could not open library 'libcurl.so': libcurl.so: cannot open shared object file: No such file or directory. Could not open library 'libcurl.so.4': libcurl.so.4: cannot open shared object file: No such file or directory' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/!
```

## references

1. [Installing Brightbox Ruby on Ubuntu](https://www.brightbox.com/docs/ruby/ubuntu)
1. [Adding Jekyll Theme to Github Pages](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/)
1. [Jekyll Theme: Cayman](https://github.com/pages-themes/cayman)
1. [Jekyll Readme: Writing Posts](https://jekyllrb.com/docs/posts/)
