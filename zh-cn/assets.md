---
layout: article
language: 'zh-cn'
version: '4.0'
---
**This article reflects v3.4 and has not yet been revised**

<a name='overview'></a>

# Assets Management

`Phalcon\Assets` is a component that allows you to manage static resources such as CSS stylesheets or JavaScript libraries in a web application.

[Phalcon\Assets\Manager](api/Phalcon_Assets_Manager) is available in the services container, so you can add resources from any part of the application where the container is available.

<a name='add'></a>

## Adding Resources

Assets supports two built-in resources: CSS and JavaScripts. You can create other resources if you need. The assets manager internally stores two default collections of resources - one for JavaScript and another for CSS.

You can easily add resources to these collections like follows:

```php
<?php

use Phalcon\Mvc\Controller;

class IndexController extends Controller
{
    public function index()
    {
        // Add some local CSS resources
        $this->assets->addCss('css/style.css');
        $this->assets->addCss('css/index.css');

        // And some local JavaScript resources
        $this->assets->addJs('js/jquery.js');
        $this->assets->addJs('js/bootstrap.min.js');
    }
}
```

Then in a view, these resources can be printed:

```php
<html>
    <head>
        <title>Some amazing website</title>

        <?php $this->assets->outputCss(); ?>
    </head>

    <body>
        <!-- ... -->

        <?php $this->assets->outputJs(); ?>
    </body>
<html>
```

Volt syntax:

```volt
<html>
    <head>
        <title>Some amazing website</title>

        {% raw %}{{ assets.outputCss() }}{% endraw %}
    </head>

    <body>
        <!-- ... -->

        {% raw %}{{ assets.outputJs() }}{% endraw %}
    </body>
<html>
```

For better page load performance, it is recommended to place JavaScript at the end of the HTML instead of in the `<head>`.

<a name='local-remote'></a>

## Local/Remote resources

Local resources are those who are provided by the same application and they're located in the document root of the application. URLs in local resources are generated by the `url` service, usually [Phalcon\Mvc\Url](api/Phalcon_Mvc_Url).

Remote resources are those such as common libraries like [jQuery](https://jquery.com), [Bootstrap](https://getbootstrap.com), etc. that are provided by a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network).

The second parameter of `addCss()` and `addJs()` says whether the resource is local or not (`true` is local, `false` is remote). By default, the assets manager will assume the resource is local:

```php
<?php

public function indexAction()
{
    // Add some remote CSS resources
    $this->assets->addCss('//netdna.bootstrapcdn.com/twitter-bootstrap/2.4.0/css/bootstrap-combined.min.css', false);

    // Then add some local CSS resources
    $this->assets->addCss('css/style.css', true);
    $this->assets->addCss('css/extra.css');
}
```

<a name='collections'></a>

## Collections

Collections group resources of the same type. The assets manager implicitly creates two collections: `css` and `js`. You can create additional collections to group specific resources to make it easier to place those resources in the views:

```php
<?php

// Javascripts in the header
$headerCollection = $this->assets->collection('header');

$headerCollection->addJs('js/jquery.js');
$headerCollection->addJs('js/bootstrap.min.js');

// Javascripts in the footer
$footerCollection = $this->assets->collection('footer');

$footerCollection->addJs('js/jquery.js');
$footerCollection->addJs('js/bootstrap.min.js');
```

Then in the views:

```php
<html>
    <head>
        <title>Some amazing website</title>

        <?php $this->assets->outputJs('header'); ?>
    </head>

    <body>
        <!-- ... -->

        <?php $this->assets->outputJs('footer'); ?>
    </body>
<html>
```

Volt syntax:

```twig
<html>
    <head>
        <title>Some amazing website</title>

        {% raw %}{{ assets.outputCss('header') }}{% endraw %}
    </head>

    <body>
        <!-- ... -->

        {% raw %}{{ assets.outputJs('footer') }}{% endraw %}
    </body>
<html>
```

<a name='url-prefixes'></a>

## URL Prefixes

Collections can be URL-prefixed, this enables you to easily change from one server to another at any moment:

```php
<?php

$footerCollection = $this->assets->collection('footer');

if ($config->environment === 'development') {
    $footerCollection->setPrefix('/');
} else {
    $footerCollection->setPrefix('http:://cdn.example.com/');
}

$footerCollection->addJs('js/jquery.js');
$footerCollection->addJs('js/bootstrap.min.js');
```

A chainable syntax is available too:

```php
<?php

$headerCollection = $assets
    ->collection('header')
    ->setPrefix('https://cdn.example.com/')
    ->setLocal(false)
    ->addJs('js/jquery.js')
    ->addJs('js/bootstrap.min.js');
```

<a name='minification-filtering'></a>

## Minification/Filtering

`Phalcon\Assets` provides built-in minification of JavaScript and CSS resources. You can create a collection of resources instructing the Assets Manager which ones must be filtered and which ones must be left as they are. In addition to the above, `Jsmin` by [Douglas Crockford](https://www.crockford.com) is part of the core extension offering minification of JavaScript files for maximum performance. In the CSS land, `CSSMin` by [Ryan Day](https://github.com/soldair) is also available to minify CSS files.

The following example shows how to minify a collection of resources:

```php
<?php

$manager

    // These JavaScripts are located in the page's bottom
    ->collection('jsFooter')

    // The name of the final output
    ->setTargetPath('final.js')

    // The script tag is generated with this URI
    ->setTargetUri('production/final.js')

    // This is a remote resource that does not need filtering
    ->addJs('code.jquery.com/jquery-1.10.0.min.js', false, false)

    // These are local resources that must be filtered
    ->addJs('common-functions.js')
    ->addJs('page-functions.js')

    // Join all the resources in a single file
    ->join(true)

    // Use the built-in Jsmin filter
    ->addFilter(
        new Phalcon\Assets\Filters\Jsmin()
    )

    // Use a custom filter
    ->addFilter(
        new MyApp\Assets\Filters\LicenseStamper()
    );
```

A collection can contain JavaScript or CSS resources but not both. Some resources may be remote, that is, they're obtained by HTTP from a remote source for further filtering. It is recommended to convert the external resources to local for better performance.

As seen above, the `addJs()` method is used to add resources to the collection, the second parameter indicates whether the resource is external or not and the third parameter indicates whether the resource should be filtered or left as is:

```php
<?php

// These Javascripts are located in the page's bottom
$jsFooterCollection = $manager->collection('jsFooter');

// This a remote resource that does not need filtering
$jsFooterCollection->addJs('code.jquery.com/jquery-1.10.0.min.js', false, false);

// These are local resources that must be filtered
$jsFooterCollection->addJs('common-functions.js');
$jsFooterCollection->addJs('page-functions.js');
```

Filters are registered in the collection, multiple filters are allowed, content in resources are filtered in the same order as filters were registered:

```php
<?php

// Use the built-in Jsmin filter
$jsFooterCollection->addFilter(
    new Phalcon\Assets\Filters\Jsmin()
);

// Use a custom filter
$jsFooterCollection->addFilter(
    new MyApp\Assets\Filters\LicenseStamper()
);
```

Note that both built-in and custom filters can be transparently applied to collections. The last step is to decide if all the resources in the collection must be joined into a single file or serve each of them individually. To tell the collection that all resources must be joined you can use the `join()` method.

If resources are going to be joined, we need also to define which file will be used to store the resources and which URI will be used to show it. These settings are set up with `setTargetPath()` and `setTargetUri()`:

```php
<?php

$jsFooterCollection->join(true);

// The name of the final file path
$jsFooterCollection->setTargetPath('public/production/final.js');

// The script HTML tag is generated with this URI
$jsFooterCollection->setTargetUri('production/final.js');
```

<a name='builtin-filters'></a>

### Built-In Filters

Phalcon provides 2 built-in filters to minify both JavaScript and CSS, their C-backend provide the minimum overhead to perform this task:

| Filter                                                                | 描述                                                                                                           |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| [Phalcon\Assets\Filters\Jsmin](api/Phalcon_Assets_Filters_Jsmin)   | Minifies JavaScript by removing unnecessary characters that are ignored by Javascript interpreters/compilers |
| [Phalcon\Assets\Filters\Cssmin](api/Phalcon_Assets_Filters_Cssmin) | Minifies CSS by removing unnecessary characters that are already ignored by browsers                         |

<a name='custom-filters'></a>

### Custom Filters

In addition to the built-in filters, you can create your own filters. These can take advantage of existing and more advanced tools like [YUI](https://yui.github.io/yuicompressor/), [Sass](https://sass-lang.com/), [Closure](https://developers.google.com/closure/compiler/), etc.:

```php
<?php

use Phalcon\Assets\FilterInterface;

/**
 * Filters CSS content using YUI
 *
 * @param string $contents
 * @return string
 */
class CssYUICompressor implements FilterInterface
{
    protected $options;

    /**
     * CssYUICompressor constructor
     *
     * @param array $options
     */
    public function __construct(array $options)
    {
        $this->options = $options;
    }

    /**
     * Do the filtering
     *
     * @param string $contents
     *
     * @return string
     */
    public function filter($contents)
    {
        // Write the string contents into a temporal file
        file_put_contents('temp/my-temp-1.css', $contents);

        system(
            $this->options['java-bin'] .
            ' -jar ' .
            $this->options['yui'] .
            ' --type css ' .
            'temp/my-temp-file-1.css ' .
            $this->options['extra-options'] .
            ' -o temp/my-temp-file-2.css'
        );

        // Return the contents of file
        return file_get_contents('temp/my-temp-file-2.css');
    }
}
```

Usage:

```php
<?php

// Get some CSS collection
$css = $this->assets->get('head');

// Add/Enable the YUI compressor filter in the collection
$css->addFilter(
    new CssYUICompressor(
        [
            'java-bin'      => '/usr/local/bin/java',
            'yui'           => '/some/path/yuicompressor-x.y.z.jar',
            'extra-options' => '--charset utf8',
        ]
    )
);
```

In a previous example, we used a custom filter called `LicenseStamper`:

```php
<?php

use Phalcon\Assets\FilterInterface;

/**
 * Adds a license message to the top of the file
 *
 * @param string $contents
 *
 * @return string
 */
class LicenseStamper implements FilterInterface
{
    /**
     * Do the filtering
     *
     * @param string $contents
     * @return string
     */
    public function filter($contents)
    {
        $license = '/* (c) 2015 Your Name Here */';

        return $license . PHP_EOL . PHP_EOL . $contents;
    }
}
```

<a name='custom-output'></a>

## Custom Output

The `outputJs()` and `outputCss()` methods are available to generate the necessary HTML code according to each type of resources. You can override this method or print the resources manually in the following way:

```php
<?php

use Phalcon\Tag;

$jsCollection = $this->assets->collection('js');

foreach ($jsCollection as $resource) {
    echo Tag::javascriptInclude(
        $resource->getPath()
    );
}
```

<a name='improving-performance'></a>

## Improving performance

There are many ways to optimize the processing resources. We'll describe a simple method below which allows to handle resourses directly through web server to optimize the response time.

First we need to set up the Assets Manager. We'll use base controller, but you can use the service provider or any other place:

```php
<?php

namespace App\Controllers;

use Phalcon\Mvc\Controller;
use Phalcon\Assets\Filters\Jsmin;

/**
 * App\Controllers\ControllerBase
 *
 * This is the base controller for all controllers in the application.
 */
class ControllerBase extends Controller
{
    public function onConstruct()
    {
        $this->assets
            ->useImplicitOutput(false)
            ->collection('global')
            ->addJs('https://code.jquery.com/jquery-4.0.1.js', false, true)
            ->addFilter(new Jsmin());
    }
}
```

Then we have to configure the routing:

```php
<?php
/*
 * Define custom routes.
 * This file gets included in the router service definition.
 */
$router = new Phalcon\Mvc\Router();

$router->addGet('/assets/(css|js)/([\w.-]+)\.(css|js)', [
    'controller' => 'assets',
    'action'     => 'serve',
    'type'       => 1,
    'collection' => 2,
    'extension'  => 3,
]);

// Other routes...
```

Finally, we need to create a controller to handle resource requests:

```php
<?php

namespace App\Controllers;

use Phalcon\Http\Response;

/**
 * Serve site assets.
 */
class AssetsController extends ControllerBase
{
    public function serveAction() : Response
    {
        // Getting a response instance
        $response = new Response();

        // Prepare output path
        $collectionName = $this->dispatcher->getParam('collection');
        $extension      = $this->dispatcher->getParam('extension');
        $type           = $this->dispatcher->getParam('type');
        $targetPath     = "assets/{$type}/{$collectionName}.{$extension}";

        // Setting up the content type
        $contentType = $type == 'js' ? 'application/javascript' : 'text/css';
        $response->setContentType($contentType, 'UTF-8');

        // Check collection existence
        if (!$this->assets->exists($collectionName)) {
            return $response->setStatusCode(404, 'Not Found');
        }

        // Setting up the Assets Collection
        $collection = $this->assets
            ->collection($collectionName)
            ->setTargetUri($targetPath)
            ->setTargetPath($targetPath);

        // Store content to the disk and return fully qualified file path
        $contentPath = $this->assets->output($collection, function (array $parameters) {
            return BASE_PATH . '/public/' . $parameters[0];
        }, $type);

        // Set the content of the response
        $response->setContent(file_get_contents($contentPath));

        // Return the response
        return $response;
    }
}
```

If precompiled resources exist in the file system they must be served directly by web server. So to get the benefit of static resources we have to update our server configuration. We will use an example configuration for Nginx. For Apache it will be a little different:

```nginx
location ~ ^/assets/ {
    expires 1y;
    add_header Cache-Control public;
    add_header ETag "";

    # If the file exists as a static file serve it directly without
    # running all the other rewrite tests on it
    try_files $uri $uri/ @phalcon;
}

location / {
    try_files $uri $uri/ @phalcon;
}

location @phalcon {
    rewrite ^(.*)$ /index.php?_url=$1;
}

# Other configuration
```

We need to create `assets/js` and `assets/css` directories in the document root of the application (eg. `public`).

Every time when the user requests resources using address of type `/assets/js/global.js` the request will be redirected to `AssetsController` in case this file is absent in the filesystem. Otherwise the resource will be handled by the web server.

It isn't the best example. However, it reflects the main idea: the reasonable configuration of a web server with an application can help optimize response time multifold.

Learn more about the Web Server Setup and Routing in their dedicated articles [Web Server Setup](/4.0/en/webserver-setup) and [Routing](/4.0/en/routing).