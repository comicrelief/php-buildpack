# Comic Relief PHP Buildpack

Forked from [Cloud Foundry PHP Buildpack](https://github.com/cloudfoundry/php-buildpack). There are more usage notes etc. in [the main repo's readme](https://github.com/cloudfoundry/php-buildpack/blob/master/README.md).

## Why?

We've forked the CF buildpack in order to support additional PHP extensions. Neither the CF nor Heroku options support everything we need as of May 2016.

## Is an extension already supported?

* See [`manifest.yml`](manifest.yml)
* Find your PHP version
* Check the `modules` key for your required extension

## Steps to support an extension

* In this buildpack:
    * Open `lib/compile_helpers.py` and append your extension name to the supported list in `validate_php_extensions()`
    * Commit and push to master before your next build
* In your app:
    * add the extension's name to `.bp-config/options.json` in `PHP_EXTENSIONS`
    * add the `.so` file in `.php-extensions`
    * if there are additional library dependencies, e.g. `/usr/lib/x86_64-linux-gnu/libgearman.so`, put them in `.php-extension-support`
    * if it's the first custom extension:
        * add a folder in `.extensions` with the name of the extension
        * inside it, add a file called `extension.py` with contents [like these](https://github.com/comicrelief/frost-service-layer/blob/a16f40dd7ccbea25c583d58068f1b034bdac06ad/.extensions/amqp-beta/extension.py) - N.B. if you take this approach and load all PHP extensions, you only need one CF extension for your app

### Getting a compiled extension to use

Need a `.so` file to use?

* Get a working local Docker image using a similar environment to the Cloud Foundry one, e.g. Ubuntu 14.04 LTS, with a Dockerfile like [this one](https://github.com/comicrelief/frost-docker/blob/master/Dockerfile) which loads your extension
* Run Docker with it
* Work out the location of the loaded PHP extensions in your image, e.g. using `phpinfo()` to find the active extensions directory
* Use `docker cp` to get the compiled `.so` file out to your local system

## Keeping this fork current

* If you've not already, track the upstream from your local: `git remote add upstream https://github.com/cloudfoundry/php-buildpack.git`
* `git fetch upstream`
* while on your local `master` branch, `git merge upstream/master`

## Things to avoid

* Don't put your extension as an `ext-*` dependency in `composer.json`. This will break the buildpack compile, because the composer CF extension is compiled before the custom extensions are, and Composer will attempt to enforce the requirement while it's not yet met.
* Don't list extensions in an app-specific `php.ini` - this is used by some other CF flavours but appears to do nothing in our use case.

## Links

* [CF PHP buildpack docs](http://docs.cloudfoundry.org/buildpacks/php/index.html)
* [PHP Bluemix example](https://developer.ibm.com/messaging/2014/07/01/using-php-mq-light-bluemix/): uses different CF flavour & buildpack but references a [simple PHP extension example](https://github.com/ibm-messaging/mqlight-sentiment-sample/blob/PHP-worker/worker.php/.extensions/cproton/extension.py)
* [WordPress PHP app CF extension example](https://github.com/dmikusa-pivotal/cf-ex-wordpress)
