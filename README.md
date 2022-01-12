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

Deployment
------------
Once you merge your changes to `main` they will be automatically deployed via [GitHub Actions](https://github.com/Formguru/docs/actions).

Acknowledgements
------------
This documentation is powered by [slatedocs](https://github.com/slatedocs/slate).
