This repository houses the public-facing documentation for the [Guru Platform](https://formguru.fitness).

Setup
------------

Start by making sure you have rbenv installed on your system:
```bash
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
```
Then inspect `.ruby-version` in the root of this repo, to find the version we are using, and install it:
```bash
rbenv install 2.7.x
```
And now finally install the packages:
```bash
bundle install
```

Development
------------
You can start a server locally by running:
```bash
bundle exec middleman server
```
which will start the server on http://localhost:4567

If you are on an M1 Mac and you see an error like this:

    /Library/Ruby/Gems/2.6.0/gems/nokogiri-1.12.5-x86_64-darwin/lib/nokogiri/extension.rb:7:in `require_relative': dlopen(/Library/Ruby/Gems/2.6.0/gems/nokogiri-1.12.5-x86_64-darwin/lib/nokogiri/2.6/nokogiri.bundle, 0x0009): tried: '/Library/Ruby/Gems/2.6.0/gems/nokogiri-1.12.5-x86_64-darwin/lib/nokogiri/2.6/nokogiri.bundle' (mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64e')), '/usr/lib/nokogiri.bundle' (no such file) - /Library/Ruby/Gems/2.6.0/gems/nokogiri-1.12.5-x86_64-darwin/lib/nokogiri/2.6/nokogiri.bundle (LoadError)


Re-install nokogiri as described in [this StackOverflow
answer](https://stackoverflow.com/a/69817978/895769).

Deployment
------------
Once you merge your changes to `main` they will be automatically deployed via [GitHub Actions](https://github.com/Formguru/docs/actions).

Acknowledgements
------------
This documentation is powered by [slatedocs](https://github.com/slatedocs/slate).
