---
date: 2020-12-31
title: 'Joomla 4.x-Tutorial - Entwicklung von Erweiterungen - Ansicht nach Kategorien'
template: post
thumbnail: '../../thumbnails/joomla.png'
slug: joomla-ansicht-nach-kategorien
langKey: de
categories:
  - JoomladE
  - Code
tags:
  - CMS
  - Joomla
---

Warum Kategorien verwenden? Oft werden Kategorien eingesetzt, wenn es viele Beiträge auf einer Site gibt. Mithilfe von Kategorien können diese einfacher gruppiert und verwaltet werden. Beispiel: In der Beitragsverwaltung können die Beiträge nach Kategorien gefiltert werden. Wenn es 200 Beiträge auf der Site gibt, ist es einfach, einen Beitrag zu finden, wenn man seine Kategorie kennt.<!-- \index{Kategorien!Frontend} -->

Für das Frontend gibt es in Joomla! eingebaute Menüpunkttypen, die Kategorien verwenden: Kategorie Blog und Kategorie Liste. Die Menüeintragstypen oder Layouts vereinfachen die Anzeige der Beiträge in einer Kategorie. Wenn ein neuer Beitrag der Kategorie zugewiesen wird, erscheint er automatisch auf der Seite. Diese Anzeige ist konfigurierbar. Beispiel: Stelle dir ein Blog-Layout der Kategorie `Events` vor, welches die neuesten Beiträge als erstes auf der Website anzeigt. Wenn ein neuer Beitrag zu dieser Kategorie hinzugefügt wird, erscheint dieser automatisch ganz oben im Blog Events. Alles, was du tun musst ist, den Beitrag zur Kategorie hinzuzufügen. Die Kategorie-Struktur, beispielsweise `Events | Onlineveranstaltungen | Sport | Yoga`, ist völlig unabhängig von der Menüstruktur der Site. Die Site kann eine oder sechs Menüebenen haben und `Yoga` kann als Menüpunkt in der ersten Ebene eingeordnet werden.

> Für Ungeduldige: Sieh dir den geänderten Programmcode in der [Diff-Ansicht](https://github.com/astridx/boilerplate/compare/t25...t26)[^github.com/astridx/boilerplate/compare/t25...t26] an und übernimm diese Änderungen in deine Entwicklungsversion.

## Schritt für Schritt

[Kategorien](https://docs.joomla.org/Category/de)[^docs.joomla.org/category/de] sind eine Möglichkeit, Inhalte in Joomla zu organisieren. Eine Kategorie enthält Elemente und andere Kategorien. Ein Artikel kann nur zu einer Kategorie gehören. Wenn eine Kategorie in einer anderen enthalten ist, ist sie eine Unterkategorie dieser Kategorie. Kommt es in deiner Struktur vor, dass einzelne Elemente zu mehreren Untergruppen gehören? Dann sind Kategorien nicht die richtige Wahl. Verwenden Sie in diesem Fall Schlagwörter oder Tags.

### Neue Dateien

<!-- prettier-ignore -->
#### [components/com\_foos/ src/Model/CategoryModel.php](https://github.com/astridx/boilerplate/compare/t25...t26#diff-71b6dccdcef138d4aabf575d418deb76)

Die Klasse, mit der wir die Daten zur Anzeige der Kategorie-Ansicht vorbereiten, erweitert die Klasse `ListModel` in der Datei `/libraries/src/MVC/Model/ListModel.php`, genau wie die Klasse `FeaturedModel` in `components/com_foos/src/Model/FeaturedModel.php`. ListModel bietet unter anderem die Möglichkeit, die Anzeige mehrerer Elemente gleichzeitig auf einer Webseite zu handhaben, einschließlich der Unterstützung für Paginierung. Nachfolgend füge ich meinen vollständigen Code ein, der sich von `com_contact` ableitet.

[components/com_foos/ src/Model/CategoryModel.php](https://github.com/astridx/boilerplate/blob/0d8c876d2435bb1cb38a62dd9a37880df9a3e178/src/components/com_foos/src/Model/CategoryModel.php)

```php {numberLines: -2}
// https://raw.githubusercontent.com/astridx/boilerplate/t26/src/components/com_foos/src/Model/CategoryModel.php

<?php
/**
 * @package     Joomla.Site
 * @subpackage  com_foos
 *
 * @copyright   Copyright (C) 2005 - 2019 Open Source Matters, Inc. All rights reserved.
 * @license     GNU General Public License version 2 or later; see LICENSE.txt
 */

namespace FooNamespace\Component\Foos\Site\Model;

defined('_JEXEC') or die;

use Joomla\CMS\Categories\Categories;
use Joomla\CMS\Categories\CategoryNode;
use Joomla\CMS\Component\ComponentHelper;
use Joomla\CMS\Factory;
use Joomla\CMS\Helper\TagsHelper;
use Joomla\CMS\Language\Multilanguage;
use Joomla\CMS\MVC\Model\ListModel;
use Joomla\CMS\Table\Table;
use Joomla\Database\ParameterType;
use Joomla\Registry\Registry;

/**
 * Single item model for a foo
 *
 * @package     Joomla.Site
 * @subpackage  com_foos
 * @since       __BUMP_VERSION__
 */
class CategoryModel extends ListModel
{
	/**
	 * Category item data
	 *
	 * @var    CategoryNode
	 */
	protected $_item = null;

	/**
	 * Array of foos in the category
	 *
	 * @var    \stdClass[]
	 */
	protected $_articles = null;

	/**
	 * Category left and right of this one
	 *
	 * @var    CategoryNode[]|null
	 */
	protected $_siblings = null;

	/**
	 * Array of child-categories
	 *
	 * @var    CategoryNode[]|null
	 */
	protected $_children = null;

	/**
	 * Parent category of the current one
	 *
	 * @var    CategoryNode|null
	 */
	protected $_parent = null;

	/**
	 * The category that applies.
	 *
	 * @var    object
	 */
	protected $_category = null;

	/**
	 * The list of other foo categories.
	 *
	 * @var    array
	 */
	protected $_categories = null;

	/**
	 * Constructor.
	 *
	 * @param   array  $config  An optional associative array of configuration settings.
	 *
	 * @since   __BUMP_VERSION__
	 */
	public function __construct($config = [])
	{
		if (empty($config['filter_fields'])) {
			$config['filter_fields'] = [
				'id', 'a.id',
				'name', 'a.name',
				'state', 'a.state',
				'ordering', 'a.ordering',
				'featuredordering', 'a.featured'
			];
		}

		parent::__construct($config);
	}

	/**
	 * Method to get a list of items.
	 *
	 * @return  mixed  An array of objects on success, false on failure.
	 */
	public function getItems()
	{
		// Invoke the parent getItems method to get the main list
		$items = parent::getItems();

		if ($items === false) {
			return false;
		}

		// Convert the params field into an object, saving original in _params
		for ($i = 0, $n = count($items); $i < $n; $i++) {
			$item = &$items[$i];

			if (!isset($this->_params)) {
				$item->params = new Registry($item->params);
			}

			// Some contexts may not use tags data at all, so we allow callers to disable loading tag data
			if ($this->getState('load_tags', true)) {
				$this->tags = new TagsHelper;
				$this->tags->getItemTags('com_foos.foo', $item->id);
			}
		}

		return $items;
	}

	/**
	 * Method to build an SQL query to load the list data.
	 *
	 * @return  string    An SQL query
	 *
	 * @since   __BUMP_VERSION__
	 */
	protected function getListQuery()
	{
		$user   = Factory::getUser();
		$groups = $user->getAuthorisedViewLevels();

		// Create a new query object.
		$db    = $this->getDbo();
		$query = $db->getQuery(true);

		$query->select($this->getState('list.select', 'a.*'))
			->select($this->getSlugColumn($query, 'a.id', 'a.alias') . ' AS slug')
			->select($this->getSlugColumn($query, 'c.id', 'c.alias') . ' AS catslug')
			->from($db->quoteName('#__foos_details', 'a'))
			->leftJoin($db->quoteName('#__categories', 'c') . ' ON c.id = a.catid')
			->whereIn($db->quoteName('a.access'), $groups);

		// Filter by category.
		if ($categoryId = $this->getState('category.id')) {
			$query->where($db->quoteName('a.catid') . ' = :acatid')
				->whereIn($db->quoteName('c.access'), $groups);
			$query->bind(':acatid', $categoryId, ParameterType::INTEGER);
		}

		// Filter by state
		$state = $this->getState('filter.published');

		if (is_numeric($state)) {
			$query->where($db->quoteName('a.published') . ' = :published');
			$query->bind(':published', $state, ParameterType::INTEGER);
		} else {
			$query->whereIn($db->quoteName('c.published'), [0,1,2]);
		}

		// Filter by start and end dates.
		$nowDate = Factory::getDate()->toSql();

		if ($this->getState('filter.publish_date')) {
			$query->where('(' . $db->quoteName('a.publish_up')
				. ' IS NULL OR ' . $db->quoteName('a.publish_up') . ' <= :publish_up)')
				->where('(' . $db->quoteName('a.publish_down')
					. ' IS NULL OR ' . $db->quoteName('a.publish_down') . ' >= :publish_down)')
				->bind(':publish_up', $nowDate)
				->bind(':publish_down', $nowDate);
		}

		// Filter by search in title
		$search = $this->getState('list.filter');

		if (!empty($search)) {
			$search = '%' . trim($search) . '%';
			$query->where($db->quoteName('a.name') . ' LIKE :name ');
			$query->bind(':name', $search);
		}

		// Filter on the language.
		if ($language = $this->getState('filter.language')) {
			$language = [Factory::getLanguage()->getTag(), '*'];
			$query->whereIn($db->quoteName('a.language'), $language);
		}

		// Set sortname ordering if selected
		if ($this->getState('list.ordering') === 'sortname') {
			$query->order($db->escape('a.sortname1') . ' ' . $db->escape($this->getState('list.direction', 'ASC')))
				->order($db->escape('a.sortname2') . ' ' . $db->escape($this->getState('list.direction', 'ASC')))
				->order($db->escape('a.sortname3') . ' ' . $db->escape($this->getState('list.direction', 'ASC')));
		} else if ($this->getState('list.ordering') === 'featuredordering') {
			$query->order($db->escape('a.featured') . ' DESC')
				->order($db->escape('a.ordering') . ' ASC');
		} else {
			$query->order($db->escape($this->getState('list.ordering', 'a.ordering')) . ' ' . $db->escape($this->getState('list.direction', 'ASC')));
		}

		return $query;
	}

	/**
	 * Method to auto-populate the model state.
	 *
	 * Note. Calling getState in this method will result in recursion.
	 *
	 * @param   string  $ordering   An optional ordering field.
	 * @param   string  $direction  An optional direction (asc|desc).
	 *
	 * @return  void
	 *
	 * @since   __BUMP_VERSION__
	 */
	protected function populateState($ordering = null, $direction = null)
	{
		$app = Factory::getApplication();
		$params = ComponentHelper::getParams('com_foos');

		// Get list ordering default from the parameters
		if ($menu = $app->getMenu()->getActive()) {
			$menuParams = $menu->getParams();
		} else {
			$menuParams = new Registry;
		}

		$mergedParams = clone $params;
		$mergedParams->merge($menuParams);

		// List state information
		$format = $app->input->getWord('format');

		$numberOfFoosToDisplay = $mergedParams->get('foos_display_num');

		if ($format === 'feed') {
			$limit = $app->get('feed_limit');
		} else if (isset($numberOfFoosToDisplay)) {
			$limit = $numberOfFoosToDisplay;
		} else {
			$limit = $app->getUserStateFromRequest('global.list.limit', 'limit', $app->get('list_limit'), 'uint');
		}

		$this->setState('list.limit', $limit);

		$limitstart = $app->input->get('limitstart', 0, 'uint');
		$this->setState('list.start', $limitstart);

		// Optional filter text
		$itemid = $app->input->get('Itemid', 0, 'int');
		$search = $app->getUserStateFromRequest('com_foos.category.list.' . $itemid . '.filter-search', 'filter-search', '', 'string');
		$this->setState('list.filter', $search);

		$orderCol = $app->input->get('filter_order', $mergedParams->get('initial_sort', 'ordering'));

		if (!in_array($orderCol, $this->filter_fields)) {
			$orderCol = 'ordering';
		}

		$this->setState('list.ordering', $orderCol);

		$listOrder = $app->input->get('filter_order_Dir', 'ASC');

		if (!in_array(strtoupper($listOrder), ['ASC', 'DESC', ''])) {
			$listOrder = 'ASC';
		}

		$this->setState('list.direction', $listOrder);

		$id = $app->input->get('id', 0, 'int');
		$this->setState('category.id', $id);

		$user = Factory::getUser();

		if ((!$user->authorise('core.edit.state', 'com_foos')) && (!$user->authorise('core.edit', 'com_foos'))) {
			// Limit to published for people who can't edit or edit.state.
			$this->setState('filter.published', 1);

			// Filter by start and end dates.
			$this->setState('filter.publish_date', true);
		}

		$this->setState('filter.language', Multilanguage::isEnabled());

		// Load the parameters.
		$this->setState('params', $params);
	}

	/**
	 * Method to get category data for the current category
	 *
	 * @return  object  The category object
	 *
	 * @since   __BUMP_VERSION__
	 */
	public function getCategory()
	{
		if (!is_object($this->_item)) {
			$app = Factory::getApplication();
			$menu = $app->getMenu();
			$active = $menu->getActive();

			if ($active) {
				$params = $active->getParams();
			} else {
				$params = new Registry;
			}

			$options = [];
			$options['countItems'] = $params->get('show_cat_items', 1) || $params->get('show_empty_categories', 0);
			$categories = Categories::getInstance('Foos', $options);
			$this->_item = $categories->get($this->getState('category.id', 'root'));

			if (is_object($this->_item)) {
				$this->_children = $this->_item->getChildren();
				$this->_parent = false;

				if ($this->_item->getParent()) {
					$this->_parent = $this->_item->getParent();
				}

				$this->_rightsibling = $this->_item->getSibling();
				$this->_leftsibling = $this->_item->getSibling(false);
			} else {
				$this->_children = false;
				$this->_parent = false;
			}
		}

		return $this->_item;
	}

	/**
	 * Get the parent category.
	 *
	 * @return  mixed  An array of categories or false if an error occurs.
	 */
	public function getParent()
	{
		if (!is_object($this->_item)) {
			$this->getCategory();
		}

		return $this->_parent;
	}

	/**
	 * Get the sibling (adjacent) categories.
	 *
	 * @return  mixed  An array of categories or false if an error occurs.
	 */
	public function &getLeftSibling()
	{
		if (!is_object($this->_item)) {
			$this->getCategory();
		}

		return $this->_leftsibling;
	}

	/**
	 * Get the sibling (adjacent) categories.
	 *
	 * @return  mixed  An array of categories or false if an error occurs.
	 */
	public function &getRightSibling()
	{
		if (!is_object($this->_item)) {
			$this->getCategory();
		}

		return $this->_rightsibling;
	}

	/**
	 * Get the child categories.
	 *
	 * @return  mixed  An array of categories or false if an error occurs.
	 */
	public function &getChildren()
	{
		if (!is_object($this->_item)) {
			$this->getCategory();
		}

		return $this->_children;
	}

	/**
	 * Generate column expression for slug or catslug.
	 *
	 * @param   \JDatabaseQuery  $query  Current query instance.
	 * @param   string           $id     Column id name.
	 * @param   string           $alias  Column alias name.
	 *
	 * @return  string
	 *
	 * @since   __BUMP_VERSION__
	 */
	private function getSlugColumn($query, $id, $alias)
	{
		return 'CASE WHEN '
			. $query->charLength($alias, '!=', '0')
			. ' THEN '
			. $query->concatenate([$query->castAsChar($id), $alias], ':')
			. ' ELSE '
			. $query->castAsChar($id) . ' END';
	}

	/**
	 * Increment the hit counter for the category.
	 *
	 * @param   integer  $pk  Optional primary key of the category to increment.
	 *
	 * @return  boolean  True if successful; false otherwise and internal error set.
	 *
	 * @since   __BUMP_VERSION__
	 */
	public function hit($pk = 0)
	{
		$input = Factory::getApplication()->input;
		$hitcount = $input->getInt('hitcount', 1);

		if ($hitcount) {
			$pk = (!empty($pk)) ? $pk : (int) $this->getState('category.id');

			$table = Table::getInstance('Category');
			$table->load($pk);
			$table->hit($pk);
		}

		return true;
	}
}

```

<!-- prettier-ignore -->
#### [components/com\_foos/ src/Service/Category.php](https://github.com/astridx/boilerplate/compare/t25...t26#diff-931af94e5b12bab015c84906dc961848)

Im `Category`-Service für den Frontend Teil setzen wir die spezifischen Optionen für unsere Komponente.

[components/com_foos/ src/Service/Category.php](https://github.com/astridx/boilerplate/blob/0d8c876d2435bb1cb38a62dd9a37880df9a3e178/src/components/com_foos/src/Service/Category.php)<!-- \index{Service!Kategorie} -->

```php {numberLines: -2}
// https://raw.githubusercontent.com/astridx/boilerplate/t26/src/components/com_foos/src/Service/Category.php

<?php
/**
 * @package     Joomla.Site
 * @subpackage  com_foos
 *
 * @copyright   Copyright (C) 2005 - 2019 Open Source Matters, Inc. All rights reserved.
 * @license     GNU General Public License version 2 or later; see LICENSE.txt
 */

namespace FooNamespace\Component\Foos\Site\Service;

\defined('_JEXEC') or die;

use Joomla\CMS\Categories\Categories;

/**
 * Foo Component Category Tree
 *
 * @since  __BUMP_VERSION__
 */
class Category extends Categories
{
	/**
	 * Class constructor
	 *
	 * @param   array  $options  Array of options
	 *
	 * @since   __BUMP_VERSION__
	 */
	public function __construct($options = [])
	{
		$options['table']      = '#__foos_details';
		$options['extension']  = 'com_foos';
		$options['statefield'] = 'published';

		parent::__construct($options);
	}
}

```

<!-- prettier-ignore -->
#### [components/com\_foos/ src/View/Category/HtmlView.php](https://github.com/astridx/boilerplate/compare/t25...t26#diff-d5d0bc03614ed16454bf9941dc8ebd7a)

Die Category-Ansicht im Frontend handhaben wir über die Datei `components/com_foos/ src/View/Category/HtmlView.php`.

[components/com_foos/ src/View/Category/HtmlView.php](https://github.com/astridx/boilerplate/blob/0d8c876d2435bb1cb38a62dd9a37880df9a3e178/src/components/com_foos/src/View/Category/HtmlView.php)

```php {numberLines: -2}
// https://github.com/astridx/boilerplate/raw/t26/src/components/com_foos/src/View/Category/HtmlView.php

<?php
/**
 * @package     Joomla.Site
 * @subpackage  com_foos
 *
 * @copyright   Copyright (C) 2005 - 2019 Open Source Matters, Inc. All rights reserved.
 * @license     GNU General Public License version 2 or later; see LICENSE.txt
 */

namespace FooNamespace\Component\Foos\Site\View\Category;

\defined('_JEXEC') or die;

use Joomla\CMS\MVC\View\CategoryView;
use FooNamespace\Component\Foos\Site\Helper\RouteHelper;

/**
 * HTML View class for the Foos component
 *
 * @since  __BUMP_VERSION__
 */
class HtmlView extends CategoryView
{
	/**
	 * @var    string  The name of the extension for the category
	 * @since  __BUMP_VERSION__
	 */
	protected $extension = 'com_foos';

	/**
	 * @var    string  Default title to use for page title
	 * @since  __BUMP_VERSION__
	 */
	protected $defaultPageTitle = 'COM_FOO_DEFAULT_PAGE_TITLE';

	/**
	 * @var    string  The name of the view to link individual items to
	 * @since  __BUMP_VERSION__
	 */
	protected $viewName = 'foo';

	/**
	 * Run the standard Joomla plugins
	 *
	 * @var    boolean
	 * @since  __BUMP_VERSION__
	 */
	protected $runPlugins = true;

	/**
	 * Execute and display a template script.
	 *
	 * @param   string  $tpl  The name of the template file to parse; automatically searches through the template paths.
	 *
	 * @return  mixed  A string if successful, otherwise an Error object.
	 */
	public function display($tpl = null)
	{
		parent::commonCategoryDisplay();

		$this->pagination->hideEmptyLimitstart = true;

		foreach ($this->items as $item) {
			$item->slug = $item->id;
			$temp = $item->params;
			$item->params = clone $this->params;
			$item->params->merge($temp);
		}

		return parent::display($tpl);
	}

	/**
	 * Prepares the document
	 *
	 * @return  void
	 */
	protected function prepareDocument()
	{
		parent::prepareDocument();

		$menu = $this->menu;
		$id = (int) @$menu->query['id'];

		if ($menu && (!isset($menu->query['option']) || $menu->query['option'] != $this->extension || $menu->query['view'] == $this->viewName
			|| $id != $this->category->id)) {
			$path = [['title' => $this->category->title, 'link' => '']];
			$category = $this->category->getParent();

			while ((!isset($menu->query['option']) || $menu->query['option'] !== 'com_foos' || $menu->query['view'] === 'foo'
				|| $id != $category->id) && $category->id > 1) {
				$path[] = ['title' => $category->title, 'link' => RouteHelper::getCategoryRoute($category->id, $category->language)];
				$category = $category->getParent();
			}

			$path = array_reverse($path);

			foreach ($path as $item) {
				$this->pathway->addItem($item['title'], $item['link']);
			}
		}

		parent::addFeed();
	}
}

```

<!-- \index{slug} -->

<!-- prettier-ignore -->
#### [components/com\_foos/ tmpl/category/default.php](https://github.com/astridx/boilerplate/compare/t25...t26#diff-3ab5c99a856218c1f3a99d1a70c97dd5)

Das wir für die Kategorie-Ansicht ebenfalls ein Template erstellen, ist nicht neu. Wie üblich legen wir die Datei `default.php` im Verzeichnis `components/com_foos/tmpl/category` an. Wir nutzen `joomla.content.category_default`. Diese Layoutdatei findest du unter `layouts/joomla/content/category_default.php`.

[components/com_foos/ tmpl/category/default.php](https://github.com/astridx/boilerplate/blob/0d8c876d2435bb1cb38a62dd9a37880df9a3e178/src/components/com_foos/tmpl/category/default.php)

```php {numberLines: -2}
// https://github.com/astridx/boilerplate/raw/t26/src/components/com_foos/tmpl/category/default.php

<?php
/**
 * @package     Joomla.Site
 * @subpackage  com_foos
 *
 * @copyright   Copyright (C) 2005 - 2019 Open Source Matters, Inc. All rights reserved.
 * @license     GNU General Public License version 2 or later; see LICENSE.txt
 */

\defined('_JEXEC') or die;

use Joomla\CMS\Layout\LayoutHelper;
?>

<div class="com-foo-category">
	<?php
		$this->subtemplatename = 'items';
		echo LayoutHelper::render('joomla.content.category_default', $this);
	?>
</div>

```

<!-- prettier-ignore -->
#### [components/com\_foos/ tmpl/category/default.xml](https://github.com/astridx/boilerplate/compare/t25...t26#diff-3e8d54f4dcfed8bbd899db937bdf3c29)

In order to be able to create a menu item for navigation in the frontend in a user-friendly way, we create the file `components/com_foos/tmpl/category/default.xml`. We have done this here in the text before several times. For example for an element or for the view of the featured entries.

[components/com_foos/ tmpl/category/default.xml](https://github.com/astridx/boilerplate/blob/0d8c876d2435bb1cb38a62dd9a37880df9a3e178/src/components/com_foos/tmpl/category/default.xml)

```xml {numberLines: -2}
<!-- https://github.com/astridx/boilerplate/raw/t26/src/components/com_foos/tmpl/category/default.xml -->

<?xml version="1.0" encoding="utf-8"?>
<metadata>
	<layout title="COM_FOOS_CATEGORY_VIEW_DEFAULT_TITLE">
		<help
			key = "JHELP_MENUS_MENU_ITEM_FOO_CATEGORY"
		/>
		<message>
			<![CDATA[COM_FOOS_CATEGORY_VIEW_DEFAULT_DESC]]>
		</message>
	</layout>

	<!-- Add fields to the request variables for the layout. -->
	<fields name="request">
		<fieldset
			name="request"
			addfieldprefix="Joomla\Component\Categories\Administrator\Field"
			>
			<field
				name="id"
				type="modal_category"
				label="JGLOBAL_CHOOSE_CATEGORY_LABEL"
				extension="com_foos"
				required="true"
				select="true"
				new="true"
				edit="true"
				clear="true"
			/>
		</fieldset>
	</fields>
	<fields name="params">
		<fieldset name="basic" label="JGLOBAL_FIELDSET_DISPLAY_OPTIONS">
			<field
				name="show_pagination"
				type="list"
				label="JGLOBAL_PAGINATION_LABEL"
				useglobal="true"
				>
				<option value="0">JHIDE</option>
				<option value="1">JSHOW</option>
				<option value="2">JGLOBAL_AUTO</option>
			</field>

			<field
				name="show_pagination_results"
				type="list"
				label="JGLOBAL_PAGINATION_RESULTS_LABEL"
				useglobal="true"
				class="custom-select-color-state"
				>
				<option value="0">JHIDE</option>
				<option value="1">JSHOW</option>
			</field>
		</fieldset>
	</fields>
</metadata>

```

> Die Kategorie-Ansichten in Joomla verfügen in der Regel über eine Menge weiterer Parameter. Beispielsweise habe ich die Unterkategorien und Filter ignoriert. So bleibt das Beispiel übersichtlich. Schau das, was dir wichtig ist, in den Core-Erweiterungen nach.

> Falls dein Element nicht angezeigt wird, liegt es unter Umständen daran, dass du beim Element den Parameter `show_name` auf no gesetzt hast.

<!-- prettier-ignore -->
#### [components/com\_foos/ tmpl/category/default_items.php](https://github.com/astridx/boilerplate/compare/t25...t26#diff-d08d72ea3e911a67f9ce50b0e543a953)

Damit der Code der Kategorie-Ansicht übersichtlich ist, arbeiten wir mit Layouts. Im Template `components/com_foos/tmpl/category/default.php` verwenden wir das Layout `joomla.content.category_default`. Dieses wiederum setzt das Layout `items` voraus, welches wir in der Datei `components/com_foos/tmpl/category/default_items.php` implementieren. Auf den ersten Blick erscheint dies umständlich. In der Praxis hat es sich bewährt.

[components/com_foos/ tmpl/category/default_items.php](https://github.com/astridx/boilerplate/blob/0d8c876d2435bb1cb38a62dd9a37880df9a3e178/src/components/com_foos/tmpl/category/default_items.php)

```php {numberLines: -2}
// https://github.com/astridx/boilerplate/raw/t26/src/components/com_foos/tmpl/category/default_items.php

<?php
/**
 * @package     Joomla.Site
 * @subpackage  com_foos
 *
 * @copyright   Copyright (C) 2005 - 2019 Open Source Matters, Inc. All rights reserved.
 * @license     GNU General Public License version 2 or later; see LICENSE.txt
 */

\defined('_JEXEC') or die;

use Joomla\CMS\HTML\HTMLHelper;
use Joomla\CMS\Language\Text;
use Joomla\CMS\Router\Route;
use Joomla\CMS\Uri\Uri;
use FooNamespace\Component\Foos\Site\Helper\RouteHelper;

HTMLHelper::_('behavior.core');
?>
<div class="com-foo-category__items">
	<form action="<?php echo htmlspecialchars(Uri::getInstance()->toString()); ?>" method="post" name="adminForm" id="adminForm">
		<?php if (empty($this->items)) : ?>
			<p>
				<?php echo Text::_('JGLOBAL_SELECT_NO_RESULTS_MATCH'); ?>
			</p>
		<?php else : ?>
			<ul class="com-foo-category__list category">
				<?php foreach ($this->items as $i => $item) : ?>
					<?php if (in_array($item->access, $this->user->getAuthorisedViewLevels())) : ?>
						<li class="row cat-list-row" >

						<div class="list-title">
							<a href="<?php echo Route::_(RouteHelper::getFooRoute($item->slug, $item->catid, $item->language)); ?>">
							<?php echo $item->name; ?></a>
							<?php echo $item->event->afterDisplayTitle; ?>

							<?php echo $item->event->beforeDisplayContent; ?>
						</div>

						<?php echo $item->event->afterDisplayContent; ?>
					</li>
					<?php endif; ?>
				<?php endforeach; ?>
			</ul>
		<?php endif; ?>

			<?php if ($this->params->get('show_pagination', 2)) : ?>
			<div class="com-foo-category__counter">
				<?php if ($this->params->def('show_pagination_results', 1)) : ?>
					<p class="counter">
						<?php echo $this->pagination->getPagesCounter(); ?>
					</p>
				<?php endif; ?>

				<?php echo $this->pagination->getPagesLinks(); ?>
			</div>
			<?php endif; ?>
	</form>
</div>

```

> Die Ansicht ist nicht gestylt. Da dies ohnehin Geschmacksache - und meiner Meinung nach Aufgabe des Templates - ist, überlasse ich das Stylen dir. Ich bin der Meinung, dass die Layouts der Kategorien die Trennung von Model, View und Controller nicht beachten. Deshalb kommt es immer wieder zu Diskussionen wie im [Issue 32012](https://github.com/joomla/joomla-cms/issues/32012)[^github.com/joomla/joomla-cms/issues/32012]. Immer wieder muss entschieden werden, ob das Einfügen einer CSS-Klasse in der Ausgabe einer Komponente zu viel Abhängigkeit bringt und nur ins Template gehört oder ob nur so ein benutzerfreundliches Angebot möglich ist - bei dem die Anzahl an Intro Artikeln im Backend über eine Benutzeroberfläche bestimmbar ist.

### Geänderte Dateien

In diesem Kapitel fügen wir lediglich neue Dateien hinzu.

## Teste deine Joomla-Komponente

1. Installiere deine Komponente in Joomla Version 4, um sie zu testen:

Kopiere die Dateien im `administrator` Ordner in den `administrator` Ordner deiner Joomla 4 Installation.

Installiere deine Komponente wie in Teil eins beschrieben, nachdem du alle Dateien kopiert hast. Joomla aktualisiert bei der Installation die Namespaces für dich. Da neue Dateien hinzugekommen sind, ist dies erforderlich.

2. Erstelle einen Menüpunkt, der die Elemente einer Kategorie unserer Erweiterung anzeigt.

![Kategorien in Joomla - Menüpunkt erstellen](/images/j4x31x1.png)

3. Wechsele ins Frontend und überzeuge dich davon, dass die Elemente korrekt angezeigt werden.

![Kategorien in Joomla - Ansicht im Frontend ](/images/j4x31x2.png)
<img src="https://vg08.met.vgwort.de/na/0fbaf7567a834f6ba69c9d31c4b529c5" width="1" height="1" alt="">
