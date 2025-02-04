# ScopedController

ScopedController is a Laravel package that will allows you to execute scopes over an Eloquent model depending on the parameters you are receiving on the URL.

Inspired on the Ruby on Rails gem [Has Scope](https://github.com/plataformatec/has_scope).

# Install

Just include it in your composer.json file

```
composer require petrelli/scoped-controller
```

Or add:

```
"petrelli/scoped-controller": "0.9.*"
```

And run `composer update`.


# Usage

Let's imagine we have a `Book` model and we want to create a filter for them.

1. Create your controller and be sure to inherit from `Petrelli\ScopedController\BaseController`.

2. Define the `$scopes` variable as an Array following the pattern `[ URLparameter => scopeName, .... ]`

```php
// Here as an example we have two scopes: year and author.
// They will be called if we receive the parameters
// byYear and byAuthor respectively.
protected $scopes = [
    'byYear'   => 'year',
    'byAuthor' => 'author',
];
```

3. Define your `$entity` variable. Usually an Eloquent Model.

```php
protected $entity = Book::class;
```

4. Get the collection and perform the call. You can use any of the available functions for your chain. If using Eloquent, you could use `get()`, or `paginate(...)`.

```php
$items = $this->collection()->get();
$items = $this->collection()->paginate(static::PER_PAGE);

```

## In action

This setup will work the following way:

```
# Get books filtered by year = 2018
/books?byYear=2018

# Get books filtered by author = cortazar
/books?byAuthor=cortazar

# Get books filtered by year and author
/books?byYear=1961&byAuthor=borges

```

# Extra functionality

## Scopes with multi-value parameters

If you need to define a multi-value parameter just pass it as an array and define the scopes as the following:


```php
// Here as an example we have two scopes: year and author.
// They will be called if we receive the parameters
// byYear and byAuthor respectively.
protected $scopes = [
    'byYear'   => 'year',
    'byAuthor' => 'author',
    'sortBy'   => ['sort_by' ['field', 'direction']],
];
```

Defining the scope as an array will allow you to pass multiple parameters to it coming from the URL as arrays.


## In action


```
# Get books filtered by year = 2018 and sorted by author in alphabetical order
/books?byYear=2018&sortBy['field']=author&sortBy['direction']=asc

# Get books filtered by author = borges and sorted by date
/books?byAuthor=borges&sortBy['field']=date&sortBy['direction']=desc
```

You get the idea. The scope is nothing but a simple two parameter element.

```
public function scopeSortBy($query, $value, $direction = 'asc')
{
    //...
}
```

Of course you can generalize to use any number of parameters. Simply add it to the $scopes definition.



## Redefine the chain

If you want to be more specific about where to execute your scopes you can always redefine  `beginOfAssociationChain()`.


For example:


```php
protected function beginOfAssociationChain()
{
    return Book::published()->where('library', 'NYC');
}
```

Here your chain will always execute before anything the `published()` scope, and also will filter books showing only the ones at the NYC library.

Then of course just use `get` or `paginate` as usual:

```php
$items = $this->collection()->get();
$items = $this->collection()->paginate(static::PER_PAGE);

```

## Applying scopes manually

You can always apply all scopes manually if you want better control:

```php

// Controller function
public function index()
{
    $items = $this->applyScopes(Book::query())->get();
}

```

## Check if there's a filter present

Usually for SEO you want to check if you have any scope present. There's a very simple function for this:

```php

// Returns true/false
$this->hasAnyScope()

```

## Use specific scopes for different actions

As the result is a Query Builder, you could keep chaining methods and scopes as you would normally do with Eloquent.

Let's take the following two controller actions:


```php

public function index()
{
    // Return Books triggering defined scopes
    $items = $this->collection()->get();
}

public function indexBestSellers()
{
    // Return Books triggering defined scopes + bestSeller scope
    $items = $this->collection()->bestSeller()->get();
}
```


# License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
