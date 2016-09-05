---

layout: rad.twig
title: Filter & Search

---

## Add Category Title Search

In every grid page, there is a search field to search database fields, currently we can only search `title` and `alias`.
Let's add foreign fields search to articles page.

Open `Form/articles/GridDefinition.php` and add two new options in search group. It means we told model to search `category.title` and
`category.alias` two more fields.

``` php
// src/Blog/Admin/Articles/GridDefinition.php

// ...

class GridDefinition extends AbstractFieldDefinition
{
	public function doDefine(Form $form)
	{
		$this->group('search', function (Form $form)
		{
			// Search Field
			$form->list('field')
				->label(Translator::translate('phoenix.grid.search.field.label'))
				->set('display', false)
				->defaultValue('*')
                ->option(Translator::translate('phoenix.core.all'), '*')
                ->option(Translator::translate('admin.article.field.title'), 'article.title')
                ->option(Translator::translate('admin.article.field.alias'), 'article.alias');

				// Add these two lines
				->option('Category Title', 'category.title')
				->option('Category Alias', 'category.alias');

			// ...
		});

		// ...
```

Back to articles page and search some keyword.

![Imgur](http://i.imgur.com/jzxOAiz.jpg)

You can see the category field is able to search. Let's add simple highlight in `ArticlesHtmlView`.

``` php
// src/Blog/Admin/View/Articles/ArticlesHtmlView.php

// ...

class ArticlesHtmlView extends GridView
{
	// ...

	protected function prepareScripts()
	{
		// ...

		// Add Highlight script
		JQueryScript::highlight('.grid-table', trim($this->data->state['input.search.content']));
	}
```

This is highlight result.

![Imgur](http://i.imgur.com/dyKjxkr.jpg)

## Add More Filters

Currently, articles page has only one filter: `state`, we can add a category filter and a date filter to help us manage articles.

### Add Category Filter

``` php
// src/Blog/Admin/Articles/GridDefinition.php

// ...

class GridDefinition extends AbstractFieldDefinition
{
		// ...

		$this->group('filter', function(Form $form)
		{
			// ...

			// Category
			$form->add('category.id', new CategoryListField)
				->label('Category')
				->option('', '')
				->option('-- Select Category --', '')
				->set('onchange', 'this.form.submit()');

		// ...
```

This is the result that we can filter by category ID.

![Imgur](http://i.imgur.com/R0v5R20.jpg)

### Add Date Filter

We add `Start Date` and `End Date` to filter a time period.

``` php
// src/Blog/Admin/Articles/GridDefinition.php

// ...

class GridDefinition extends AbstractFieldDefinition
{
       use PhoenixFieldTrait;

		// ...

		$this->group('filter', function(Form $form)
		{
			// ...

			// Category
			$form->add('category.id', new CategoryListField)
                ->label('Category')
                ->option('', '')
                ->option('-- Select Category --', '')
                ->set('onchange', 'this.form.submit()');

			// Start Date
			$this->add('start_date', new CalendarField)
                		->label('Start Date')
                		->set('format', 'YYYY-MM-DD')
                		->set('placeholder', 'Start Date');

			// End Date
			$form->add('end_date', new CalendarField)
				->label('End Date')
				->set('format', 'YYYY-MM-DD')
				->set('placeholder', 'End Date');

		// ...
```

The start and end date use `>=` and `<=` to filter items, not `=`, so we must configure it in `ArticlesModel`.

``` php

class ArticlesModel extends ListModel
{
	// ...

	// This is an important security setting.
	// Add these two fields that will able to pass the fields checking.
	// Otherwise anyone can try your database.
	protected $allowFields = array(
		'start_date', 'end_date'
	);

	// ...

	protected function configureFilters(FilterHelperInterface $filterHelper)
	{
		// Override start_date action
		$filterHelper->setHandler('start_date', function(Query $query, $field, $value)
		{
			if ((string) $value !== '')
			{
				$query->where('article.created >= ' . $query->quote($value));
			}
		});

		// Override end_date action
		$filterHelper->setHandler('end_date', function(Query $query, $field, $value)
		{
			if ((string) $value !== '')
			{
				$query->where('article.created <= ' . $query->quote($value));
			}
		});
	}
```

OK, you can try to filter the date.

![Imgur](http://i.imgur.com/jLsnier.jpg)

Go to debugger, you will see the SQL is:

``` sql
SELECT
-- ...

WHERE article.created >= '1970-09-10' AND article.created <= '1973-09-09'

-- ...
```
