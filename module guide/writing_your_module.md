# Writing your module

## Autoloading

Before we start writing our first lines of code, a word about the naming of your files, classes, labels, ... If you name all your files well, you'll never have to include, require or embed a single file, Fork CMS will do this for you.

### camelCasing

All your classes, variables, id's... in your php, js and css-files, should be named using camelCasing. Names will exists only out of letters and numbers (there are no spaces) but every new word starts with an uppercase letter. Some examples:

* *new message* becomes *newMessage*
* *a great class* becomes *aGreatClass*
* *variable* becomes *variable*
* *mini blog* becomes *miniBlog*

Your files and tables & columns in the database however, should never be written in camelCasing, but always in lowercase and an underscore instead of a space. The examples above become:

* *new message* becomes *new_message*
* *a great class* becomes *a_great_class*
* *variable* becomes *variable*
* *mini blog* becomes mini_blog*

### Javascript

Not only will your .php files be autoloaded, you can autoload js-files too. There are two possibilities.

You name your file like the module, e.g. mini_blog.js and put it in the js-folder in your module folder. This way, the js-file will be included on every page of the module. This is what we're doing for our mini-blog module because we need the same javascript code on the detail and the index page.

Should a certain peace of js-code only be used on one action, then you give it the name of that action, e.g. detail.js.

### Templates

As you have have seen in our example, the module folder contains a layout folder which contains the templates and widgets to display the pages. These templates can be considered a default. When a new theme is applied to the website, the template for every module can be overwritten to better match the theme's style.

To avoid altering files in the module folder (so you can re-use it later without having to think about altered code), you can create your own theme which will have a folder in the themes folder: /frontend/themes/the_a_theme/modules/. Here you put a folder with the module name (mini_blog) which contains the templates, widgets, css, ... folders, which in turn contain the custom files.

When Fork CMS opens a webpage, it first searches for the requested template in the /frontend/themes/<selected_theme_in_fork_cms>/modules/modulename/layout/templates folder. If a template with the correct name (e.g. detail.tpl) is not found, Fork CMS falls back to the template in the default folder, frontend/modules/modulename/layout/templates.

> Backend templates
> In contrast to the frontend, the templates for the backend actions are always located in “backend/modules/modulename/layout/templates”. Themes can only be applied to the frontend, so the overwrite rules don't apply to the backend.

## Model

As you perhaps have noticed, the only PHP-file we didn't cover in the previous chapter is the model.php file, perhaps the most important one.

This file contains all the generic functions, most of the time functions that make a connection to the database.

A rule of thumb should be that every line of (parametrized!) SQL you'll write, has to be written in a model.php file.

Beneath you'll find some lines of the model.php file of the mini blog module. We'll use this example to illustrate some conventions.

```
class FrontendMiniBlogModel implements FrontendTagsInterface
{
    ...

    /**
     * Fetch one article based upon it's meta url
     *
     * @return array
     * @param string $URL The URL for the item.
     */
    public static function get($URL)
    {
        return (array) FrontendModel::getDB()->getRecord('SELECT i.id, i.language, i.title, i.introduction, i.text,
            UNIX_TIMESTAMP(i.edited) AS edited, i.user_id,
            m.keywords AS meta_keywords, m.keywords_overwrite AS meta_keywords_overwrite,
            m.description AS meta_description, m.description_overwrite AS meta_description_overwrite,
            m.title AS meta_title, m.title_overwrite AS meta_title_overwrite, m.url
            FROM mini_blog AS i
            INNER JOIN meta AS m ON i.meta_id = m.id
            WHERE i.language = ? AND i.publish = ? AND m.url = ?
            LIMIT 1',
            array(FRONTEND_LANGUAGE, 'Y', (string) $URL));
    }

    ... class continues for another 200 lines
}
```

First we encounter the naming of the class. We can split this in three parts. Mind that classnames always begin with an uppercase and the rest follows the general in camelCasing.

1. The application, because we'll have a Backend model too (*Frontend*)
2. The module (*MiniBlog*)
3. The action/name of the file (*Model*)

It can be a good idea to write your model.php first. The first thing you have to do is figure out what exactly every action has to be able to do.

* detail
	* get all the details of one article
	* get the previous and next article (for navigation)
* index
	* get a list of items, ordered by date, grouped by a given number and offset
	* get the total amount of articles (for navigation)
* recent_posts
	* get a list of items, ordered by date, grouped by a given number
* this_is_awesome
	* check if an article exists
	* add 1 to the number of reports of the article

As you are a programmer you'll probably have seen that we want to fetch a list of items twice, in almost the same way. Instead of writing this code twice for both actions, we write it in model.php where it's shared between the actions (and even other modules.)

Notice that the tags interface got implemented to make sure you create the propper functions that are used by the tags module.

> If you check the code in model.php, you'll see we have some other methods too concerning implementing tags and searching, ... we'll discuss these later.

## Config

Another required file is the config, place it in the root of your backend and frontend module folder.

```
<?php
class FrontendMiniBlogConfig extends FrontendBaseConfig
{
	/**
	 * The default action
	 *
	 * @var	string
	 */
	protected $defaultAction = 'index';

	/**
	 * The disabled actions
	 *
	 * @var	array
	 */
	protected $disabledActions = array();
}
```