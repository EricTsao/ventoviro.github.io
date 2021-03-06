---
layout: documentation.twig
title: URI and Route Building

---

## Base URI

Sometimes you may put your application at sub folder of a domain, Windwalker provides a uri data help you get base uri.

If our application located at `http://domain.com/sites/windwalker/` and we open `/flower/sakura?foo=bar` page, we may use this code to get uri data:

``` php
$uri = \Windwalker\Ioc::getUriData();

// Get Base path
echo $uri->current;
echo $uri->full;
echo $uri->path;
echo $uri->root;
echo $uri->host;
echo $uri->route;
```

We will get:

```
current  ---> http://domain.com/sites/windwalker/flower/sakura
full     ---> http://domain.com/sites/windwalker/flower/sakura?foo=bar
path     ---> /sites/windwalker
root     ---> http://domain.com/sites/windwalker
host     ---> http://domain.com
route    ---> flower/sakura
```

These uri data can help us add base path to our uri, so the link will not break if
we use relative url.

Add host and base path.

``` php
echo $link = $uri->root . '/romeo/and/juliet';

// OR

echo $link = $uri->root() . '/romeo/and/juliet';
```

The output will be:

```
http://domain.com/sites/windwalker/romeo/and/juliet
```

Or only add base path by `path` variable.

``` php
echo $uri->path . '/flower/sakura';

// OR

echo $uri->path() . '/flower/sakura';
```

We'll get a uri start from root, so the relative path will not break.

``` html
/sites/windwalker/flower/sakura
```

### Use Method Call

``` php
$uri->root('flower/sakura');
$uri->path('flower/sakura');
```

### Use UriData in View

View has already included uri object as a global variable.

``` html
<link href="<?php echo $uri->path; ?>/asset/css/bootstrap.min.css">

OR

<link href="<?php echo $uri->path('asset/css/bootstrap.min.css'); ?>">
```

The output will be:

``` html
<link href="/sites/windwalker/media/css/bootstrap.min.css">
```

Start with `/` means this url will based on domain host.

## Build Route

Every route in Windwalker has a key, we called it **route name** or **route profile**, this name will help us quickly build route.

For example, this is the routing file:

``` yaml
flower_page:
    pattern: /flower/page/(id)-(alias).html
    controller: Flower\Controller\Page
```

Then we can build this route by `Router::route()`

``` php
use Windwalker\Core\Router;

// Note: don't use \Windwalker\Router\Router
echo Router::route('flower_page', ['id' => 25, 'alias' => 'foo-bar-baz']);
```

The output will be:

``` html
flower/page/25-foo-bar-baz.html
```

This is a very useful function that you can change roue name but won't worry of link will be broke.

### Build Package Route

If your routes is definded in a package, you must add package alias before route name, and separate by at (`@`):

``` php
use Windwalker\Core\Router;

Router::route('flower@sakuras', array('page' => $page));
```

Or use PackageRouter, we can ignore package prefix, Package will auto add it:

``` php
$package = PackageHelper::getPackage('flower');

// No necessary to add 'flower:'
$package->router->route('sakuras', array('page' => $page));

// If you want to build rote in other package, you can re-add package name
$package->router->route('user@login');
```

### Encode URL in Template

If we print URL in HTML, we must encode some special chars.

``` php
// In php engine
// URL: /flower/sakuras?foo=bar&baz=yoo

<?php echo $this->escape(Router::route('flower@sakuras', ['foo' => 'bar', 'baz' => 'yoo'])); ?>

// OR in Edge

{{ $this->escape(Router::route('flower@sakuras', ['foo' => 'bar', 'baz' => 'yoo'])) }}
```

The printed URL will be:

```
/flower/sakuras?foo=bar&amp;baz=yoo
```

### Relative or Absolute URL

Router has 3 mode, `RAW`, `PATH` or `FULL`:

``` php
echo Router::route('flower@sakuras', array(), Router::TYPE_RAW);
echo Router::route('flower@sakuras', array(), Router::TYPE_PATH);
echo Router::route('flower@sakuras', array(), Router::TYPE_FULL);
```

Result:

``` html
flower/sakuras
/sites/windwalker/flower/sakuras
http://domain.com/sites/windwalker/flower/sakuras
```

The RAW route used to store in database, the PATH route used to print in HTML, the FULL route used to redirect.

### Build Route in Controller

If you are in controller, we can use PackageRouter to build route, this way is more safer because we can auto get current package routes.

``` php
// Original way, we have to know package name
use Windwalker\Core\Router;

$route = Router::route('flower@sakura');

// Simpler way to let package handle it
$route = $this->package->router->route('sakura');

$this->setRedirect($route);
```

### Build Route in View Template

View has also includes package router, just build route like this:

``` html
<a href="<?php echo $router->route('sakura', ['id' => 25]); ?>">Link</a>
```

In Blade or Edge

``` php
<a href="{{ $router->route('sakura', ['id' => 25]) }}">Link</a>

OR

@route('sakura', ['id' => 25]))
```

Twig

``` php
<a href="{{ router.route('sakura', ['id' => 25]) }}">Link</a>
```
