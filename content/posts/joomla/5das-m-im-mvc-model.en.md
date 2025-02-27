---
date: 2020-12-05
title: 'Joomla 4.x Tutorial - Extension Development - The M in MVC: Model'
template: post
thumbnail: '../../thumbnails/joomla.png'
slug: en/das-m-im-mvc-model
langKey: en
categories:
  - JoomlaEn
  - Code
tags:
  - CMS
  - Joomla
---

No new functionality is added in this part. We improve the previous structure. Each web application consists of

- Logic,
- data and
- the presentation.

It is problematic to combine these three elements in one class. Especially for larger projects. Joomla uses the [Model-View-Controller-Concept (MVC)](https://en.wikipedia.org/wiki/Model_View_Controller)[^en.wikipedia.org/wiki/model_view_controller]. In this tutorial part, we add a Model to the frontend. The Model object is responsible for the data and its processing.<!-- \index{Model-View-Controller} -->

> For impatient people: View the changed program code in the [Diff View](https://github.com/astridx/boilerplate/compare/t3...t4)[^github.com/astridx/boilerplate/compare/t3...t4] and copy these changes into your development version.

## Step by step

### New files

<!-- prettier-ignore -->
#### [components/com\_foos/ src/Model/FooModel.php](https://github.com/astridx/boilerplate/compare/t3...t4#diff-599caddf64a6ed0c335bc9c9f828f029)

With the model it is also so that you do not reinvent the wheel. You extend the Joomla class `BaseDatabaseModel`. Then implement only what you specifically use. In our case it is the output `$this->message = 'Hello Foo!';` for which we create the method `getMsg()`.

[components/com_foos/ src/Model/FooModel.php](https://github.com/astridx/boilerplate/blob/4951c642c75d353de06bcc78de3efb7e20b0f93d/src/components/com_foos/src/Model/FooModel.php)

```php {numberLines: -2}
// https://raw.githubusercontent.com/astridx/boilerplate/t4/src/components/com_foos/src/Model/FooModel.php

<?php

/**
 * @package     Joomla.Site
 * @subpackage  com_foos
 *
 * @copyright   Copyright (C) 2005 - 2020 Open Source Matters, Inc. All rights reserved.
 * @license     GNU General Public License version 2 or later; see LICENSE.txt
 */

namespace FooNamespace\Component\Foos\Site\Model;

\defined('_JEXEC') or die;

use Joomla\CMS\MVC\Model\BaseDatabaseModel;

/**
 * Foo model for the Joomla Foos component.
 *
 * @since  __BUMP_VERSION__
 */
class FooModel extends BaseDatabaseModel
{
	/**
	 * @var string message
	 */
	protected $message;

	/**
	 * Get the message
	 *
	 * @return  string  The message to be displayed to the user
	 */
	public function getMsg()
	{
		if (!isset($this->message)) {
			$this->message = 'Hello Foo!';
		}

		return $this->message;
	}
}

```

### Modified files

<!-- prettier-ignore -->
#### [components/com\_foos/ src/View/Foo/HtmlView.php](https://github.com/astridx/boilerplate/compare/t3...t4#diff-c77adeff4ff9e321c996e0e12c54b656)

We get the data of the model in the view with `$this->msg = $this->get('Msg');`. This seems complicated in this simple example. In complex applications, this procedure has proven itself. The data calculation is done in the model. The view handles the design of the data.

[components/com_foos/ src/View/Foo/HtmlView.php](https://github.com/astridx/boilerplate/blob/4951c642c75d353de06bcc78de3efb7e20b0f93d/src/components/com_foos/src/View/Foo/HtmlView.php)

```php {diff}
 	public function display($tpl = null)
 	{
+		$this->msg = $this->get('Msg');
+
 		return parent::display($tpl);
 	}
 }

```

> You may be confused by the call `$this->get('Msg');` as I was when I first started using Joomla. The method in the model is called `getMsg()`, but we call it here via `get('Msg')`. This doesn't fit. If you have dealt with object oriented programming before, you are tempted to call it via `getMsg()`. If you are using Joomla, you will have an easier time using things the way they are prepared. You call [Getter](https://en.wikipedia.org/wiki/Mutator_method)[^en.wikipedia.org/wiki/mutator_method] in the model via the method `get()` with the appropriate parameter.

<!-- prettier-ignore -->
#### [components/com\_foos/ tmpl/foo/default.php](https://github.com/astridx/boilerplate/compare/t3...t4#diff-a33732ebd6992540b8adca5615b51a1f)

We output the data via the template. Here, everything will be packed into HTML tags later.

[components/com_foos/ tmpl/foo/default.php](https://github.com/astridx/boilerplate/blob/4951c642c75d353de06bcc78de3efb7e20b0f93d/src/components/com_foos/tmpl/foo/default.php)

```php {diff}
 \defined('_JEXEC') or die;
 ?>
-Hello Foos
+
+Hello Foos: <?php echo $this->msg;

```

## Test your Joomla component

1. install your component in Joomla version 4 to test it: Copy the files in the `administrator` folder into the `administrator` folder of your Joomla 4 installation. Copy the files in the `components` folder into the `components` folder of your Joomla 4 installation. A new installation is not necessary. Continue using the files from the previous part.

2. look at the frontend view of your component. Make sure that the data for the output is generated by the model.

![Joomla - Usind the Model - Frontend view](/images/j4x5x1.png)
<img src="https://vg08.met.vgwort.de/na/d91b51e606454390a2dc4f6c776b837b" width="1" height="1" alt="">
