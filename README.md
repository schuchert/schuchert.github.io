After cloning the repo, you'll need to
~~~ bash
sudo apt install -y ruby ruby-dev libpng-dev -y
~~~

Then in the home directory:
~~~ bash
_bin/serve
~~~ bash

This does the following for you:
~~~ bash
bundle install
bundle exec jekyll serve --incremental
~~~
