---
layout: article
language: 'zh-cn'
version: '4.0'
title: 'Phalcon\Loader'
---
# Class **Phalcon\Loader**

*implements* [Phalcon\Events\EventsAwareInterface](Phalcon_Events_EventsAwareInterface)

[源码在GitHub](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/loader.zep)

This component helps to load your project classes automatically based on some conventions

```php
<?php

use Phalcon\Loader;

// Creates the autoloader
$loader = new Loader();

// Register some namespaces
$loader->registerNamespaces(
    [
        "Example\Base"    => "vendor/example/base/",
        "Example\Adapter" => "vendor/example/adapter/",
        "Example"          => "vendor/example/",
    ]
);

// Register autoloader
$loader->register();

// Requiring this class will automatically include file vendor/example/adapter/Some.php
$adapter = new \Example\Adapter\Some();

```

## 方法

public **setEventsManager** ([Phalcon\Events\ManagerInterface](Phalcon_Events_ManagerInterface) $eventsManager)

设置事件管理器

public **getEventsManager** ()

返回内部事件管理器

public **setExtensions** (*array* $extensions)

Sets an array of file extensions that the loader must try in each attempt to locate the file

public **getExtensions** ()

Returns the file extensions registered in the loader

public **registerNamespaces** (*array* $namespaces, [*mixed* $merge])

Register namespaces and their related directories

public **setFileCheckingCallback** (*mixed* $callback = null): [Phalcon\Loader](Phalcon_Loader)

Sets the file check callback.

```php
<?php

// Default behavior.
$loader->setFileCheckingCallback("is_file");

// Faster than `is_file()`, but implies some issues if
// the file is removed from the filesystem.
$loader->setFileCheckingCallback("stream_resolve_include_path");

// Do not check file existence.
$loader->setFileCheckingCallback(null);
```

A [Phalcon\Loader\Exception](Phalcon_Loader_Exception) is thrown if the $callback parameter is not a `callable` or `null`;

protected **prepareNamespace** (*array* $namespace)

...

public **getNamespaces** ()

Returns the namespaces currently registered in the autoloader

public **registerDirs** (*array* $directories, [*mixed* $merge])

Register directories in which "not found" classes could be found

public **getDirs** ()

Returns the directories currently registered in the autoloader

public **registerFiles** (*array* $files, [*mixed* $merge])

Registers files that are "non-classes" hence need a "require". This is very useful for including files that only have functions

public **getFiles** ()

Returns the files currently registered in the autoloader

public **registerClasses** (*array* $classes, [*mixed* $merge])

Register classes and their locations

public **getClasses** ()

Returns the class-map currently registered in the autoloader

public **register** ([*mixed* $prepend])

Register the autoload method

public **unregister** ()

Unregister the autoload method

public **loadFiles** ()

Checks if a file exists and then adds the file by doing virtual require

public **autoLoad** (*mixed* $className)

Autoloads the registered classes

public **getFoundPath** ()

Get the path when a class was found

public **getCheckedPath** ()

Get the path the loader is checking for a path