# nexgen-inquiry buildpack

This is a [custom Heroku buildpack](https://devcenter.heroku.com/articles/buildpack-api) intended for the nexgen-inquiry app. It will probably not work for other apps without further modification.

It was originally based on [heroku-buildpack-ember-cli](https://github.com/tonycoco/heroku-buildpack-ember-cli/tree/19afa2d02b2feef631f85bc0a424edf5246862f3) (this link goes to the specific ref).

Major modifications :
* The detect script was updated to look for `.embedded-ember-dir`, which specifies where Ember is.
* The compile script was updated to use `.embedded-ember-dir` to know where to compile.
* Nginx build, configuration, deployment was removed from the compile script.

Original Readme below.

# Heroku Buildpack for Ember CLI Applications

This buildpack will work out of the box with Ember CLI generated applications. It installs Node, Nginx and generates a production build with the Ember CLI.

## Usage

Creating a new Heroku instance from an Ember CLI application's parent directory:

    $ heroku create --buildpack https://github.com/tonycoco/heroku-buildpack-ember-cli.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    ...

Or, use the Heroku Deploy button:

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

## Configuration

You can set a few different environment variables to turn on features in this buildpack.

### Nginx Workers

Set the number of workers for Nginx (Default: `4`):

    $ heroku config:set NGINX_WORKERS=4

This will depend on your Heroku instance size and the amount of dynos.

### API Proxy

Set an API proxy URL:

    $ heroku config:set API_URL=http://api.example.com/

Set your API's prefix path (Default: `/api/`):

    $ heroku config:set API_PREFIX_PATH=/api/

*Trailing slashes are important. For more information about API proxies and avoiding CORS, [read this](http://oskarhane.com/avoid-cors-with-nginx-proxy_pass).*

### Authentication

Setting `BASIC_AUTH_USER` and `BASIC_AUTH_PASSWORD` in your Heroku application will activate basic authentication:

    $ heroku config:set BASIC_AUTH_USER=EXAMPLE_USER
    $ heroku config:set BASIC_AUTH_PASSWORD=EXAMPLE_PASSWORD

*Be sure to use HTTPS for added security.*

### Force HTTPS/SSL

For most Ember applications that make any kind of authenticated requests HTTPS should be used. It supports the headers `X-Forwarded-Proto` ([used by Heroku](https://devcenter.heroku.com/articles/http-routing#heroku-headers)) and `CF-Visitor` ([used by CloudFlare](https://support.cloudflare.com/hc/en-us/articles/200170536-How-do-I-redirect-HTTPS-traffic-with-Flexible-SSL-and-Apache-)). Enable this feature in Nginx by setting `FORCE_HTTPS`:

    $ heroku config:set FORCE_HTTPS=true

### Prerender.io

[Prerender.io](https://prerender.io) allows your application to be crawled by search engines.

Set the service's host and token:

    $ heroku config:set PRERENDER_HOST=service.prerender.io
    $ heroku config:set PRERENDER_TOKEN=<your-prerender-token>

*Sign up for the hosted [Prerender.io](https://prerender.io) service or host it yourself. See the [project's repo](https://github.com/prerender/prerender) for more information.*

### Naked Domain Redirection

Visitors can be redirected from your "naked domain" (`example.com`) to `www.example.com`. Set your naked domain:

    $ heroku config:set NAKED_DOMAIN=example.com

*This uses a HTTP 301 redirect to forward the request. All parameters are preserved.*

### Private Repositories

Configure a `GIT_SSH_KEY` to allow Heroku access to private repositories:

    $ heroku config:set GIT_SSH_KEY=<base64-encoded-private-key>

If present, the buildpack expects the base64 encoded contents of a private key whose public key counterpart has been registered with GitHub on an account with access to any private repositories needed by the application. Prior to executing `npm install` and `bower install` it decodes the contents into a file, launches ssh-agent and registers that keyfile. Once NPM install is finished, it cleans up the environment and file system of the key contents.

Private NPM dependency URLs must be in the form of `git+ssh://git@github.com:[user]/[repo].git`. Private Bower dependency URLs must be in the form of `git@github.com:[user]/[repo].git`. Either NPM or Bower URLs may have a trailing `#semver`.

### Environment

Choose the environment you want to build by setting:

    $ heroku config:set EMBER_ENV=production

### Before and After Hooks

Have the buildpack run your own scripts before and after the `ember build` by creating a `hooks/before_hook.sh` or `hooks/after_hook.sh` file in your Ember CLI application:

    $ mkdir -p hooks

For a before build hook:

    $ touch hooks/before_hook.sh
    $ chmod +x hooks/before_hook.sh

For an after build hook:

    $ touch hooks/after_hook.sh
    $ chmod +x hooks/after_hook.sh

*See below for examples.*

#### Example Before Hook: Compass

[Compass](http://compass-style.org) can be installed using the before build hook. Create `hooks/before_hook.sh` and add the following script:

```bash
#!/usr/bin/env bash

export GEM_HOME=$build_dir/.gem/ruby/2.2.0
export PATH=$GEM_HOME/bin:$PATH

if test -d $cache_dir/ruby/.gem; then
  status "Restoring ruby gems directory from cache"
  cp -r $cache_dir/ruby/.gem $build_dir
  HOME=$build_dir gem update compass --user-install --no-rdoc --no-ri
else
  HOME=$build_dir gem install compass --user-install --no-rdoc --no-ri
fi

rm -rf $cache_dir/ruby
mkdir -p $cache_dir/ruby

if test -d $build_dir/.gem; then
  status "Caching ruby gems directory for future builds"
  cp -r $build_dir/.gem $cache_dir/ruby
fi
```

### Custom Nginx

In your Ember CLI application, add a `config/nginx.conf.erb` file and add your own Nginx configuration.

*You should copy the existing configuration file in this repo and make changes to it for best results.*

### Caching

The Ember CLI buildpack caches your NPM and Bower dependencies by default. This is similar to the [Heroku Buildpack for Node.js](https://github.com/heroku/heroku-buildpack-nodejs). This makes typical deployments much faster. Note that dependencies like [`components/ember#canary`](http://www.ember-cli.com/#using-canary-build-instead-of-release) will not be updated on each deploy.

To [purge the cache](https://github.com/heroku/heroku-repo#purge_cache) and reinstall all dependencies, run:

    $ heroku plugins:install https://github.com/heroku/heroku-repo.git
    $ heroku repo:purge_cache -a APPNAME

## Troubleshooting

Clean your project's dependencies:

    $ npm cache clear
    $ bower cache clean
    $ rm -rf node_modules bower_components
    $ npm install --no-optional
    $ bower install

Be sure to save any Bower or NPM resolutions. Now, let's build your Ember CLI application locally:

    $ ember build

Check your `git status` and see if that process has made any changes to your application's code. Now, try your Heroku deployment again.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Contributors

A special thanks to everyone who maintains and helps out on the project!

- Aaron Chambers
- Aaron Ortbals
- Aaron Renner
- Adriaan
- Adriaan van Rossum
- Bill Curtis
- Brett Chalupa
- Chris Santero
- John Griffin
- Jonas Brusman
- Jonathan Johnson
- Jonathan Zempel
- Jordan Morano
- Juan Pablo Pinilla Ossa
- Kori Roys
- Matt McGinnis
- Optimal Cadence
- Peter Brown
- Ryan LeFevre
- Tony Coconate
- harianus
- sbl

_Generated with: `git log --format='%aN' | sort -u`._
