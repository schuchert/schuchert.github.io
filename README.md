## Setup

Install Ruby and dependencies:
~~~ bash
brew install rbenv ruby-build
rbenv init
rbenv install 3.4.5
rbenv global 3.4.5
gem install bundler
~~~

After `rbenv init`, restart your terminal or add rbenv to your shell profile.

## Run

~~~ bash
_bin/serve
~~~

This runs:
~~~ bash
bundle install
bundle exec jekyll serve --incremental
~~~
