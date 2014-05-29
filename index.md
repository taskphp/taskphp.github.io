---
layout: default
title: Pure PHP task runner. 
---

[![Build Status](https://travis-ci.org/taskphp/task.svg?branch=master)](https://travis-ci.org/taskphp/task)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/taskphp/task/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/taskphp/task/?branch=master)
[![Code Coverage](https://scrutinizer-ci.com/g/taskphp/task/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/taskphp/task/?branch=master)
[![Latest Stable Version](https://poser.pugx.org/task/task/v/stable.png)](https://packagist.org/packages/task/task)
[![Total Downloads](https://poser.pugx.org/task/task/downloads.png)](https://packagist.org/packages/task/task)
[![Latest Unstable Version](https://poser.pugx.org/task/task/v/unstable.png)](https://packagist.org/packages/task/task)
[![License](https://poser.pugx.org/task/task/license.png)](https://packagist.org/packages/task/task)

Got a PHP project? Heard of Grunt and Gulp but don't use NodeJS?  Task is a pure PHP task runner.

* Leverage PHP as a scripting language, and as your platform of choice.
* Use loads of nice features inspired by Grunt and Gulp (and Phing).
* Employ Symfony components for effortless CLI goodness.
* Extend with plugins.

Ask us anything on Twitter at [@taskphp](https://twitter.com/taskphp).

Example
=======

```php
<?php

use Task\Plugin;

require 'vendor/autoload.php';

$project = new Task\Project('wow');

$project->inject(function ($container) {
    $container['phpspec'] = new Plugin\PhpSpecPlugin;
    $container['fs'] = new Plugin\FilesystemPlugin;
    $container['sass'] = (new Plugin\Sass\ScssPlugin)
        ->setPrefix('sass');
    $container['watch'] = new Plugin\WatchPlugin;
});

$project->addTask('welcome', function () {
    $this->getOutput()->writeln('Hello!');
});

$project->addTask('test', ['phpspec', function ($phpspec) {
    $phpspec->command('run')
        ->setFormat('pretty')
        ->setVerbose(true)
        ->pipe($this->getOutput());
}]);

$project->addTask('css', ['fs', 'sass', function ($fs, $sass) {
    $fs->open('my.scss')
        ->pipe($sass)
        ->pipe($fs->touch('my.css'));
}]);

$project->addTask('css.watch', ['watch', function ($watch) use ($project) {
    $watch->init('my.scss')
        ->addListener('modify', function ($event) {
            $this->runTask('css', $this->getOutput());
        })
        ->start();
}]);

return $project;
```

Installation
============

Add to your `composer.json`:

```json
...
    "require": {
        "task/task": "~0.5"
    }
...
```

This will allow you to instantiate a `Task\Project`. To run tasks from the command line, install [task/cli](https://github.com/taskphp/cli). You should probably do this now!

Usage
=====

The only requirements are that you implement a `Taskfile` that returns a `Task\Project`:

```php
<?php

# Include the task/task library and your dependencies.
require 'vendor/autoload.php';

# Instantiate a project by giving it a name.
$project = new Task\Project('foo');

# Return the project!
return $project;
```

We suggest putting the `Taskfile` in the root of your project. The CLI package will look for a `Taskfile` in the current working directory, so `cd` in to your project and run:

```bash
$> task
foo version 

  --verbose        -v|vv|vvv Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
  --version        -V Display this application version.
  --ansi              Force ANSI output.
  --no-ansi           Disable ANSI output.
  --no-interaction -n Do not ask any interactive question.

Available commands:
  help    Displays help for a command
  list    Lists commands
  shell
```

If you've used [Symfony's Console component](https://github.com/symfony/console) before this will look familiar! Your `Task\Project` is a `Symfony\Component\Console\Application` and so you have a pretty CLI application out of the box.

Add a task:

```php
<?php

# Include the task/task library and your dependencies.
require 'vendor/autoload.php';

# Instantiate a project by giving it a name.
$project = new Task\Project('foo');

# Add a task
$project->addTask('greet', function () {
    # Write to stdout
    $this->getOutput()->writeln('Hello, World!');
});

# Return the project!
return $project;
```

As you can see, tasks are just `Closure`s. To run it:

```bash
$> task greet
Hello, World!
```

Plugins
=======

Plugins are where the real work gets done.

```json
...
    "require": {
        "task/task": "~0.1",
        "task/process": "~0.1"
    }
...
```

```php
<?php

use Task\Plugin\ProcessPlugin;

# Include the task/task library and your dependencies.
require 'vendor/autoload.php';

# Instantiate a project by giving it a name.
$project = new Task\Project('foo');

# Add your plugins to the project's DI container.
$project->inject(function ($container) {
    $container['ps'] = new ProcessPlugin;
});

# Add a task.
$project->addTask('greet', function () {
    # Write to stdout.
    $this->getOutput()->writeln('Hello, World!');
});

# Use the handy array syntax to inject plugins into tasks.
$project->addTask('whoami', ['ps', function ($ps) {
    # Use streams to pass data between plugins and interfaces.
    $ps->build('whoami')->pipe($this->getOutput());
}]);

# Return the project!
return $project;
```

```bash
$> task whoami
mbfisher
```

This is a totally pointless example but it demonstrates some core concepts.

DI
--

Dependency injection is used to setup plugins and inject them into tasks. `Project::inject` allows you to fill a `Pimple` container up with anything you like. 

Injection
---------

Plugins are injected into tasks using `Task\Injector`. Instead of a `Closure`, pass `Project::addTask` an array with your task as the last element. The preceding elements should be IDs stored in the container; they will be retrieved and passed as arguments to the task.

Streams
-------

Plugins are encouraged to use NodeJS-style streams for handling data flows. `Task\Plugin\Stream\ReadableInterface` provides `read` and `pipe` methods, `Task\Plugin\Stream\WritableInterface` provides a `write` method. In the example above `ProcessPlugin::build` returns a `Task\Plugin\Process\ProcessBuilder`, which implements `ReadableInterface`, allowing us to `pipe` a `Task\Plugin\Console\Output\Output` instance to it, which implements `WritableInterface`.

Discussion
==========

* See [nikic's article on PHP over XML](https://nikic.github.io/2012/07/09/A-plea-for-less-XML-configuration-files.html) for a great argument for using pure PHP for configuration.

Alternatives
============

There's a few PHP task runners popping up:

* Bldr - http://bldr.io/
* Robo - http://codegyre.github.io/Robo/
