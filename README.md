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

Acknowledgements
------------
This documentation is powered by [slatedocs](https://github.com/slatedocs/slate).
