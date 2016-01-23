# About

This is a [custom Heroku buildpack](https://devcenter.heroku.com/articles/buildpack-api) intended for the nexgen-inquiry app. It will probably not work for other apps without further modification.

The buildpack is mostly a copy-and-paste job, on April 21 2015, from the [heroku-buildpack-ember-cli](https://github.com/tonycoco/heroku-buildpack-ember-cli/tree/19afa2d02b2feef631f85bc0a424edf5246862f3) (this link goes to the specific ref).

Major modifications :
* The detect script was updated to look for `.embedded-ember-dir`, which specifies where Ember is.
* The compile script was updated to use `.embedded-ember-dir` to know where to compile.
* Nginx build, configuration, deployment was removed from the compile script.
