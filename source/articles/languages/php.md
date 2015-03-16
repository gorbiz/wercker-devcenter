---
sidebar_current: "languages-php"
---

# PHP

Below we go into detail on how to get started with wercker and php. You can also
find guides specific to PHP below.

## Guides

* [Getting started with PHP and wercker](/articles/languages/php/gettingstarted-php.html)

## Sample application

There is an [getting-started-php](http://github.com/wercker/getting-started-php)
application that demonstrates dependency management via Composer and running an
integration test with PHPUnit.

## PHP box

The `wercker/php` box runs on ubuntu 12.04 and provides multiple versions of
PHP. It also includes XDebug, PEAR, Pyrus, Composer and PHPUnit. For PHP
projects your [wercker.yml](/articles/werckeryml/) could look as follows:

``` yaml
box: wercker/php
build:
  steps:
    - script:
        name: Install extensions
        code: |-
          pecl install memcache
          pecl install SQLite
    - script:
        name: install dependencies
        code: |-
            composer install --no-interaction
    - script:
        name: Serve application for integration tests
        code: php -S localhost:8000 >> /dev/null &
    - script:
        name: PHPUnit integration tests
        code: phpunit --configuration phpunit.xml
```

At the top you see the 'box' definition that states we want the 'wercker/php'
box. Next, there is a 'build' clause, this defines your build pipeline on
wercker. In the `wercker.yml` above we have defined several custom steps.

## PHP versions

There are three versions available on the wercker PHP box. The previous stable
release PHP 5.4, the current stable release PHP 5.5. By default the current
stable release 5.5 is active.

```
$ php --version
PHP 5.5.20 ...
```

The wercker PHP box uses [phpenv](https://github.com/CHH/phpenv) to manage the
versions. You can switch the PHP version with the `phpenv global` command:

```
$ php --version
PHP 5.5.20 ...
$ phpenv global 5.4
$ php --version
PHP 5.4.36 ...
```

Here is an example of a script step that activates PHP 5.3:

```yaml
- script:
    name: Activate PHP 5.3
    code: phpenv global 5.3
```

## PHP configuration

As the PHP box has phpenv installed, you can use `phpenv config-add` to add a
custom configuration file:

```yaml
- script:
    name: Add project-config.ini
    code: phpenv config-add project-config.ini
```

You must have a `project-config.ini` file inside your code repository. Here is
an example of the possible contents of this file:

```
date.timezone = "Europe/Amsterdam"
```

You could also add a line to the existing configuration file:

```
echo 'date.timezone = "Europe/Amsterdam"' >> $HOME/.phpenv/versions/$(phpenv version-name)/etc/php.ini
```

## Available tools

The following package managers are installed [PEAR](http://pear.php.net/),
[Pyrus](http://pear.php.net/manual/en/pyrus.about.php) and
[Composer](http://getcomposer.org/).

### Installing Composer packages

Composer is a tool for dependency management in PHP. Composer is globally
installed on the PHP box and the version does not change when changing the PHP
version with phpenv.

Here is an example of a script step that installs all the dependencies:

```yaml
- script:
    name: install dependencies
    code: composer install
```

Composer will automatically install any packages you may have listed in the
`require-dev` section of your `composer.json`. If these are not required for
your build, things can be sped up by setting the `--no-dev` option to skip the
development dependencies:

```yaml
- script:
    name: install dependencies
    code: composer install --no-dev
```

#### Composer lock file

After installing dependencies using `composer update`, Composer writes the list
of the exact versions it installed into a `composer.lock` file. This locks the
project to those specific versions. It is a best practise to commit this
`composer.lock` along with `composer.json` into version control to ensure
wercker uses the same versions of dependencies as in your local development
environment. This also guarantees that the same versions of dependencies are
used among the full development team.

This means that if any of the dependencies get a new version, you won't get the
updates automatically. To update to the new version, use the `update` on your
local development machine and commit the updated `composer.lock` file:

```
$ php composer.phar update
$ git add composer.lock
$ git commit -m 'Updates all dependencies to new version'
```

If you only want to update one dependency:

```
$ php composer.phar update monolog/monolog
$ git add composer.lock
$ git commit -m 'Updates monolog to new version'
```

### Installing pear packages

Pear is a package manager for PHP. Pear is globally installed on the PHP box and
the version does not change when changing the PHP version with phpenv.

Here is an example of an script step that installs the `pear/PHPDoc` package with pear.

```yaml
- script:
    name: install dependencies
    code: |-
      pear install pear/PHPDoc
      phpenv rehash
```

After the install, you should refresh your path by executing a `php rehash`.

### Installing pyrus packages

Pyrus is the successor of pear and is a package manager for PHP. Pyrus is
globally installed on the PHP box and the version does not change when changing
the PHP version with phpenv.

Here is an example of an script step that installs the `pear/PHPDoc` package
with pear.

```yaml
- script:
    name: install dependencies
    code: |-
      pyrus install pear/PHPDoc
      phpenv rehash
```

After the install, you should refresh your path by executing an `php rehash`.

## Running tests with PHPUnit

PHPUnit is a test runner for PHP. PHPUnit is globally installed on the PHP box
and the version does not change when changing the PHP version with phpenv.

```yaml
- script:
    name: run unit tests
    code: phpunit
```

You need to have a `bootstrap.php` for your tests that requires the
`vendor/autoload.php` file. Here is an example of such:

```php
<?php
$file = __DIR__.'/../vendor/autoload.php';
if (!file_exists($file)) {
    throw new RuntimeException('Install dependencies to run test suite.');
}

$autoload = require_once $file;
?>
```

And you need to add this `bootstrap.php` file to your `phpunit.xml`:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<phpunit bootstrap="tests/bootstrap.php">
  <testsuites>
    <testsuite name="integration tests">
      <directory>tests</directory>
    </testsuite>
  </testsuites>
</phpunit>
```

If your tests are integration tests, you can serve your application with the
following script step:

```yaml
- script:
    name: Serve application
    code: php -S localhost:80 >> output.txt &
```

This will host your PHP application on port 80. All output will be piped to
`output.txt` to keep your log clean. The ampersand (`&`) at the end of this
command makes the command asynchronous.

## Installing extensions

The PHP box comes with PECL which can be used to compile and install extensions
to the environment. Installing an extension with PECL will automatically
activate them as well. In other words, it will update the php.ini accordingly.
Here is an example of a script step that installs `memcache` and `SQLite`
extension:

```yaml
- script:
    name: Install extensions
    code: |-
      pecl install memcache
      pecl install SQLite
```

Some extensions ask for user input during the installation. The `apc` package is
an example of this:

```
$ pecl install apc
Downloading APC-3.1.13.tgz ...
Starting to download APC-3.1.13.tgz (171,591 bytes)
.....................................done: 171,591 bytes
55 source files, building
running: phpize
Configuring for:
PHP Api Version:         20100412
Zend Module Api No:      20100525
Zend Extension Api No:   220100525
Enable internal debugging in APC [no] :
```

If this was a build on wercker it would eventually timeout. To solve this you
can print user input to the process. Here is a script step that prints a newline
(enter) to the pecl install command:

```yaml
- script:
    name: Install extensions
    code: echo '\n' | pecl install apc
```

-------

<div class="authorCredits">
    <span class="profile-picture">
        <img src="https://secure.gravatar.com/avatar/5864d682bb0da7bedf31601e4e3172e7?d=identicon&s=192" alt="Pieter Joost van de Sande"/>
    </span>
    <ul class="authorCredits">

        <!-- author info -->
        <li class="authorCredits__name">
            <h4>Pieter Joost van de Sande</h4>
            <em>
                Pieter Joost is an engineer and community manager at wercker
            </em>
        </li>

        <!-- info -->
        <li>
            <a href="http://beta.wercker.com" target="_blank">
                <i class="icon-company"></i> <em>wercker</em>
            </a>
            <a href="http://twitter.com/pjvds" target="_blank">
                <i class="icon-twitter"></i>
                <em> mies</em>
            </a>
        </li>

    </ul>
</div>

-------
##### last modified: Sept 7, 2013
-------
