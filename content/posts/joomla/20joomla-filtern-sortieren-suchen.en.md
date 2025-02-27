---
date: 2020-12-20
title: 'Joomla 4.x Tutorial - Extension Development - Filter, Sort, Search'
template: post
thumbnail: '../../thumbnails/joomla.png'
slug: en/joomla-filtern-sortieren-suchen
langKey: en
categories:
  - JoomlaEn
  - Code
tags:
  - CMS
  - Joomla
---

Filtering, sorting and searching - now we organize the Joomla 4 component ! Joomla offers view filters and search tools with which you can limit the number of visible items. If the status filter is set accordingly, only items whose status is published will be displayed. Beside the status filter the search tools offer the search by title or content and the possibility to sort the table, i.e. to change the order.<!-- \index{filtering} --><!-- \index{sorting} --><!-- \index{searching} -->

![Joomla Filter Sort and Search -Search Tools](/images/j4x20x1.png)

> For impatient people: Look at the changed program code in the [Diff view](https://github.com/astridx/boilerplate/compare/t15a...t16)[^github.com/astridx/boilerplate/compare/t15a...t16] and take over these changes into your development version.

## Step by step

### New files

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ forms/filter_foos.xml](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-680833320598887b6d6cc4feb95d4408)

First, we create the form through which the filters will be set.

[administrator/components/com_foos/ forms/filter_foos.xml](https://github.com/astridx/boilerplate/blob/t16/src/administrator/components/com_foos/forms/filter_foos.xml)

```xml {numberLines: -2}
<!-- https://raw.githubusercontent.com/astridx/boilerplate/t16/src/administrator/components/com_foos/forms/filter_foos.xml -->

<?xml version="1.0" encoding="utf-8"?>
<form>

	<fields name="filter">

		<field
			name="search"
			type="text"
			inputmode="search"
			label="COM_FOOS_FILTER_SEARCH_LABEL"
			description="COM_FOOS_FILTER_SEARCH_DESC"
			hint="JSEARCH_FILTER"
		/>

		<field
			name="featured"
			type="list"
			onchange="this.form.submit();"
			default=""
			>
			<option value="">JOPTION_SELECT_FEATURED</option>
			<option value="0">JUNFEATURED</option>
			<option value="1">JFEATURED</option>
		</field>

		<field
			name="published"
			type="status"
			label="JOPTION_SELECT_PUBLISHED"
			onchange="this.form.submit();"
			>
			<option value="">JOPTION_SELECT_PUBLISHED</option>
		</field>

		<field
			name="category_id"
			type="category"
			label="JCATEGORY"
			multiple="true"
			extension="com_foos"
			layout="joomla.form.field.list-fancy-select"
			hint="JOPTION_SELECT_CATEGORY"
			onchange="this.form.submit();"
			published="0,1,2"
		/>

		<field
			name="access"
			type="accesslevel"
			label="JOPTION_SELECT_ACCESS"
			onchange="this.form.submit();"
			>
			<option value="">JOPTION_SELECT_ACCESS</option>
		</field>

		<field
			name="language"
			type="contentlanguage"
			label="JOPTION_SELECT_LANGUAGE"
			onchange="this.form.submit();"
			>
			<option value="">JOPTION_SELECT_LANGUAGE</option>
			<option value="*">JALL</option>
		</field>

	</fields>

	<fields name="list">

		<field
			name="fullordering"
			type="list"
			label="JGLOBAL_SORT_BY"
			default="a.name ASC"
			onchange="this.form.submit();"
			>
			<option value="">JGLOBAL_SORT_BY</option>
			<option value="a.ordering ASC">JGRID_HEADING_ORDERING_ASC</option>
			<option value="a.ordering DESC">JGRID_HEADING_ORDERING_DESC</option>
			<option value="a.published ASC">JSTATUS_ASC</option>
			<option value="a.published DESC">JSTATUS_DESC</option>
			<option value="a.name ASC">JGLOBAL_TITLE_ASC</option>
			<option value="a.name DESC">JGLOBAL_TITLE_DESC</option>
			<option value="category_title ASC">JCATEGORY_ASC</option>
			<option value="category_title DESC">JCATEGORY_DESC</option>
			<option value="access_level ASC">JGRID_HEADING_ACCESS_ASC</option>
			<option value="access_level DESC">JGRID_HEADING_ACCESS_DESC</option>
			<option value="association ASC" requires="associations">JASSOCIATIONS_ASC</option>
			<option value="association DESC" requires="associations">JASSOCIATIONS_DESC</option>
			<option value="language_title ASC" requires="multilanguage">JGRID_HEADING_LANGUAGE_ASC</option>
			<option value="language_title DESC" requires="multilanguage">JGRID_HEADING_LANGUAGE_DESC</option>
			<option value="a.id ASC">JGRID_HEADING_ID_ASC</option>
			<option value="a.id DESC">JGRID_HEADING_ID_DESC</option>
		</field>

		<field
			name="limit"
			type="limitbox"
			label="JGLOBAL_LIST_LIMIT"
			default="25"
			onchange="this.form.submit();"
		/>
	</fields>
</form>

```

> `featured` is included here as a filter field for the sake of completeness, although we do not support this in the extension yet.

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ sql/updates/mysql/16.0.0.sql](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-49ec0cc94fa89db6f20d60195f94c0fe)

In case of an update of your component, the file `16.0.0.sql` adds a column to store the sequence.

[administrator/components/com_foos/ sql/updates/mysql/16.0.0.sql](https://github.com/astridx/boilerplate/blob/t16/src/administrator/components/com_foos/sql/updates/mysql/16.0.0.sql)

```xml {numberLines: -2}
-- https://raw.githubusercontent.com/astridx/boilerplate/t16/src/administrator/components/com_foos/sql/updates/mysql/16.0.0.sql

ALTER TABLE `#__foos_details` ADD COLUMN  `ordering` int(11) NOT NULL DEFAULT 0 AFTER `alias`;

```

### Modified files

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ forms/foo.xml](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-262e27353fbe755d3813ea2df19cd0ed)

The form used to create or modify an element is extended with a field for specifying the order.

[administrator/components/com_foos/ forms/foo.xml](https://github.com/astridx/boilerplate/blob/9a7f1349a8b8371a96e93056d7764c557686f7c1/src/administrator/components/com_foos/forms/foo.xml)

```xml {diff}
 			label="JFIELD_ACCESS_LABEL"
 			size="1"
 		/>
+
+		<field
+			name="ordering"
+			type="ordering"
+			label="JFIELD_ORDERING_LABEL"
+			content_type="com_foos.foo"
+		/>
 	</fieldset>
 </form>

```

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ sql/install.mysql.utf8.sql](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-896f245bc8e493f91277fd33913ef974)

In case of a new installation, the script in the file `install.mysql.utf8.sql` creates the database. Here we add a column to store the order.

[administrator/components/com_foos/ sql/install.mysql.utf8.sql](https://github.com/astridx/boilerplate/blob/9a7f1349a8b8371a96e93056d7764c557686f7c1/src/administrator/components/com_foos/sql/install.mysql.utf8.sql)

```xml {diff}

 ALTER TABLE `#__foos_details` ADD COLUMN  `language` char(7) NOT NULL DEFAULT '*' AFTER `alias`;

 ALTER TABLE `#__foos_details` ADD KEY `idx_language` (`language`);
+
+ALTER TABLE `#__foos_details` ADD COLUMN  `ordering` int(11) NOT NULL DEFAULT 0 AFTER `alias`;

```

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ src/Model/FoosModel.php](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-2daf62ad6c51630353e31eaa3cc28626)

There are a lot of changes in the model for the list. In the constructor we first save the filter fields to the configuration.

In the `getListQuery()` method we adjust the database query to respect the filters and sorting. This way the data is immediately in the form in which we display it.

[administrator/components/com_foos/ src/Model/FoosModel.php](https://github.com/astridx/boilerplate/blob/9a7f1349a8b8371a96e93056d7764c557686f7c1/src/administrator/components/com_foos/src/Model/FoosModel.php)

```php {diff}
 use Joomla\CMS\MVC\Model\ListModel;
 use Joomla\CMS\Language\Associations;
 use Joomla\CMS\Factory;
+use Joomla\Utilities\ArrayHelper;

 /**
  * Methods supporting a list of foo records.

 	 */
 	public function __construct($config = array())
 	{
+
+		if (empty($config['filter_fields']))
+		{
+			$config['filter_fields'] = array(
+				'id', 'a.id',
+				'name', 'a.name',
+				'catid', 'a.catid', 'category_id', 'category_title',
+				'published', 'a.published',
+				'access', 'a.access', 'access_level',
+				'ordering', 'a.ordering',
+				'language', 'a.language', 'language_title',
+				'publish_up', 'a.publish_up',
+				'publish_down', 'a.publish_down',
+			);
+
+			$assoc = Associations::isEnabled();
+
+			if ($assoc)
+			{
+				$config['filter_fields'][] = 'association';
+			}
+		}
+
 		parent::__construct($config);
 	}
 	/**

 				array(
 					'a.id', 'a.name', 'a.alias', 'a.access',
 					'a.catid', 'a.published', 'a.publish_up', 'a.publish_down',
-					'a.language'
+					'a.language', 'a.ordering', 'a.state'
 				)
 			)
 		);

 			$query->where($db->quoteName('a.language') . ' = ' . $db->quote($language));
 		}

+		// Filter by access level.
+		if ($access = $this->getState('filter.access'))
+		{
+			$query->where($db->quoteName('a.access') . ' = ' . (int) $access);
+		}
+
+		// Filter by published state
+		$published = (string) $this->getState('filter.published');
+
+		if (is_numeric($published))
+		{
+			$query->where($db->quoteName('a.published') . ' = ' . (int) $published);
+		}
+		elseif ($published === '')
+		{
+			$query->where('(' . $db->quoteName('a.published') . ' = 0 OR ' . $db->quoteName('a.published') . ' = 1)');
+		}
+
+		// Filter by a single or group of categories.
+		$categoryId = $this->getState('filter.category_id');
+
+		if (is_numeric($categoryId))
+		{
+			$query->where($db->quoteName('a.catid') . ' = ' . (int) $categoryId);
+		}
+		elseif (is_array($categoryId))
+		{
+			$query->where($db->quoteName('a.catid') . ' IN (' . implode(',', ArrayHelper::toInteger($categoryId)) . ')');
+		}
+
+		// Filter by search in name.
+		$search = $this->getState('filter.search');
+
+		if (!empty($search))
+		{
+			if (stripos($search, 'id:') === 0)
+			{
+				$query->where('a.id = ' . (int) substr($search, 3));
+			}
+			else
+			{
+				$search = $db->quote('%' . str_replace(' ', '%', $db->escape(trim($search), true) . '%'));
+				$query->where(
+					'(' . $db->quoteName('a.name') . ' LIKE ' . $search . ')'
+				);
+			}
+		}
+
+		// Add the list ordering clause.
+		$orderCol = $this->state->get('list.ordering', 'a.name');
+		$orderDirn = $this->state->get('list.direction', 'asc');
+
+		if ($orderCol == 'a.ordering' || $orderCol == 'category_title')
+		{
+			$orderCol = $db->quoteName('c.title') . ' ' . $orderDirn . ', ' . $db->quoteName('a.ordering');
+		}
+
+		$query->order($db->escape($orderCol . ' ' . $orderDirn));
+
 		return $query;
 	}

```

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ src/View/Foos/HtmlView.php](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-8e3d37bbd99544f976bf8fd323eb5250)

The view loads the filter form `src/administrator/components/com_foos/ forms/foo.xml`, which is displayed in the upper area. Besides we add here the check if the active user is allowed to perform actions.

[administrator/components/com_foos/ src/View/Foos/HtmlView.php](https://github.com/astridx/boilerplate/blob/9a7f1349a8b8371a96e93056d7764c557686f7c1/src/administrator/components/com_foos/src/View/Foos/HtmlView.php)

```php {diff}
 \defined('_JEXEC') or die;

+use Joomla\CMS\Component\ComponentHelper;
+use Joomla\CMS\Helper\ContentHelper;
+use Joomla\CMS\Language\Associations;
 use Joomla\CMS\Factory;
 use Joomla\CMS\Language\Text;
 use Joomla\CMS\MVC\View\HtmlView as BaseHtmlView;
diff --git a/src/administrator/components/com_foos/src/View/Foos/HtmlView.php b/src/administrator/components/com_foos/src/View/Foos/HtmlView.php
index af7c2fa6..f966bd96 100644
--- a/src/administrator/components/com_foos/src/View/Foos/HtmlView.php
+++ b/src/administrator/components/com_foos/src/View/Foos/HtmlView.php

 use Joomla\CMS\Toolbar\ToolbarHelper;
 use FooNamespace\Component\Foos\Administrator\Helper\FooHelper;
 use Joomla\CMS\Factory;
+use Joomla\CMS\MVC\View\GenericDataException;

 /**
  * View class for a list of foos.

 	 */
 	protected $items;

+	protected $state;
+
+	/**
+	public $filterForm;
+
+	public $activeFilters;
+
 	/**
 	 * The sidebar markup
 	 *

 	{
 		$this->items = $this->get('Items');

+		$this->filterForm = $this->get('FilterForm');
+		$this->activeFilters = $this->get('ActiveFilters');
+		$this->state = $this->get('State');
+
+		// Check for errors.
+		if (count($errors = $this->get('Errors')))
+		{
+			throw new GenericDataException(implode("\n", $errors), 500);
+		}
+
+		// Preprocess the list of items to find ordering divisions.
+		// TODO: Complete the ordering stuff with nested sets
+		foreach ($this->items as &$item)
+		{
+			$item->order_up = true;
+			$item->order_dn = true;
+		}
+
 		// We don't need toolbar in the modal window.
 		if ($this->getLayout() !== 'modal')
 		{

 			{
 				// If the language is forced we can't allow to select the language, so transform the language selector filter into a hidden field.
 				$languageXml = new \SimpleXMLElement('<field name="language" type="hidden" default="' . $forcedLanguage . '" />');
+				$this->filterForm->setField($languageXml, 'filter', true);
+
+				// Also, unset the active language filter so the search tools is not open by default with this filter.
+				unset($this->activeFilters['language']);
+
+				// One last changes needed is to change the category filter to just show categories with All language or with the forced language.
+				$this->filterForm->setFieldAttribute('category_id', 'language', '*,' . $forcedLanguage, 'filter');
 			}
 		}


 	 */
 	protected function addToolbar()
 	{
-		FooHelper::addSubmenu('foos');
-		$this->sidebar = \JHtmlSidebar::render();
-
 		$canDo = ContentHelper::getActions('com_foos');

```

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ tmpl/foos/default.php](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-3186af99ea4e3321b497b86fcd1cd757)

The code below shows all the essentials for using `searchtools` in the list view of the backend.
In the case of the header, I replaced `<?php echo TEXT::_('JGRID_HEADING_ACCESS') ?>` with `<?php echo HTMLHelper::_('searchtools.sort', 'JGRID_HEADING_ACCESS', 'access_level', $listDirn, $listOrder); ?>`. This way the header of the table is marked with a small arrow when a sort is active in a column.

[administrator/components/com_foos/ tmpl/foos/default.php](https://github.com/astridx/boilerplate/blob/9a7f1349a8b8371a96e93056d7764c557686f7c1/src/administrator/components/com_foos/tmpl/foos/default.php)

```php {diff}
 use Joomla\CMS\Language\Multilanguage;
 use Joomla\CMS\Language\Associations;
 use Joomla\CMS\Layout\LayoutHelper;
+use Joomla\CMS\Session\Session;

+$canChange = true;
 $assoc = Associations::isEnabled();
+$listOrder = $this->escape($this->state->get('list.ordering'));
+$listDirn  = $this->escape($this->state->get('list.direction'));
+$saveOrder = $listOrder == 'a.ordering';

+if ($saveOrder && !empty($this->items)) {
+	$saveOrderingUrl = 'index.php?option=com_foos&task=foos.saveOrderAjax&tmpl=component&' . Session::getFormToken() . '=1';
+}
 ?>
 <form action="<?php echo Route::_('index.php?option=com_foos'); ?>" method="post" name="adminForm" id="adminForm">
 	<div class="row">

 						echo 'col-md-12';
 					} ?>">
 			<div id="j-main-container" class="j-main-container">
+				<?php echo LayoutHelper::render('joomla.searchtools.default', ['view' => $this]); ?>
 				<?php if (empty($this->items)) : ?>
 					<div class="alert alert-warning">
 						<?php echo Text::_('JGLOBAL_NO_MATCHING_RESULTS'); ?>
 					</div>
 				<?php else : ?>
 					<table class="table" id="fooList">
+						<caption id="captionTable" class="sr-only">
+							<?php echo Text::_('COM_FOOS_TABLE_CAPTION'); ?>, <?php echo Text::_('JGLOBAL_SORTED_BY'); ?>
+						</caption>
 						<thead>
 							<tr>
+								<th scope="col" style="width:1%" class="text-center d-none d-md-table-cell">
+									<?php echo HTMLHelper::_('searchtools.sort', '', 'a.ordering', $listDirn, $listOrder, null, 'asc', 'JGRID_HEADING_ORDERING', 'icon-menu-2'); ?>
+								</th>
 								<td style="width:1%" class="text-center">
 									<?php echo HTMLHelper::_('grid.checkall'); ?>
 								</td>
 								<th scope="col" style="width:1%" class="text-center d-none d-md-table-cell">
-									<?php echo Text::_('COM_FOOS_TABLE_TABLEHEAD_NAME'); ?>
-								</th>
-								<th scope="col" style="width:1%; min-width:85px" class="text-center">
-									<?php echo TEXT::_('JSTATUS'); ?>
+									<?php echo HTMLHelper::_('searchtools.sort', 'COM_FOOS_TABLE_TABLEHEAD_NAME', 'a.name', $listDirn, $listOrder); ?>
 								</th>
 								<th scope="col" style="width:10%" class="d-none d-md-table-cell">
-									<?php echo TEXT::_('JGRID_HEADING_ACCESS') ?>
+									<?php echo HTMLHelper::_('searchtools.sort', 'JGRID_HEADING_ACCESS', 'access_level', $listDirn, $listOrder); ?>
 								</th>
 								<?php if ($assoc) : ?>
 									<th scope="col" style="width:10%">
-										<?php echo Text::_('COM_FOOS_HEADING_ASSOCIATION'); ?>
+										<?php echo HTMLHelper::_('searchtools.sort', 'COM_FOOS_HEADING_ASSOCIATION', 'association', $listDirn, $listOrder); ?>
 									</th>
 								<?php endif; ?>
 								<?php if (Multilanguage::isEnabled()) : ?>
 									<th scope="col" style="width:10%" class="d-none d-md-table-cell">
-										<?php echo Text::_('JGRID_HEADING_LANGUAGE'); ?>
+										<?php echo HTMLHelper::_('searchtools.sort', 'JGRID_HEADING_LANGUAGE', 'language_title', $listDirn, $listOrder); ?>
 									</th>
 								<?php endif; ?>
 								<th scope="col" style="width:1%; min-width:85px" class="text-center">
-									<?php echo Text::_('JSTATUS'); ?>
+									<?php echo HTMLHelper::_('searchtools.sort', 'JSTATUS', 'a.published', $listDirn, $listOrder); ?>
 								</th>
 								<th scope="col">
-									<?php echo Text::_('COM_FOOS_TABLE_TABLEHEAD_ID'); ?>
+									<?php echo HTMLHelper::_('searchtools.sort', 'JGRID_HEADING_ID', 'a.id', $listDirn, $listOrder); ?>
 								</th>
 							</tr>
 						</thead>

 						foreach ($this->items as $i => $item) :
 							?>
 							<tr class="row<?php echo $i % 2; ?>">
+								<td class="order text-center d-none d-md-table-cell">
+									<?php
+									$iconClass = '';
+									if (!$canChange) {
+										$iconClass = ' inactive';
+									} else if (!$saveOrder) {
+										$iconClass = ' inactive tip-top hasTooltip" title="' . HTMLHelper::_('tooltipText', 'JORDERINGDISABLED');
+									}
+									?>
+									<span class="sortable-handler<?php echo $iconClass; ?>">
+										<span class="icon-menu" aria-hidden="true"></span>
+									</span>
+									<?php if ($canChange && $saveOrder) : ?>
+										<input type="text" style="display:none" name="order[]" size="5"
+											value="<?php echo $item->ordering; ?>" class="width-20 text-area-order">
+									<?php endif; ?>
+								</td>
 								<td class="text-center">
 									<?php echo HTMLHelper::_('grid.id', $i, $item->id); ?>
 								</td>


 									<div class="small">
 										<?php echo Text::_('JCATEGORY') . ': ' . $this->escape($item->category_title); ?>
-									 </div>
+									</div>
 								</th>
 								<td class="text-center">
 									<?php
-									echo HTMLHelper::_('jgrid.published', $item->published, $i, 'foos.', true, 'cb', $item->publish_up, $item->publish_down);
+									echo HTMLHelper::_('jgrid.published', $item->published, $i, 'foos.', $canChange, 'cb', $item->publish_up, $item->publish_down);
 									?>
 								</td>
 								<td class="small d-none d-md-table-cell">

```

<!-- prettier-ignore -->
#### [administrator/components/ com\_foos/ tmpl/foos/modal.php](https://github.com/astridx/boilerplate/compare/t15a...t16#diff-aeba8d42de72372f42f890d454bf928e)

Icons show us if a column is sorted and in which direction. To make the sorting clear to someone who doesn't see these markers, we add a `<caption>` element. This is not displayed, it is read out.

> The [class `visually-hidden`](https://getbootstrap.com/docs/5.0/getting-started/accessibility/#visually-hidden-content) hides an element for all devices except screen readers.

[administrator/components/com_foos/ tmpl/foos/modal.php](https://github.com/astridx/boilerplate/blob/9a7f1349a8b8371a96e93056d7764c557686f7c1/src/administrator/components/com_foos/tmpl/foos/modal.php)

```php {diff}
 			<table class="table table-sm">
 				<thead>
 					<tr>
+					<caption id="captionTable" class="sr-only">
+						<?php echo Text::_('COM_FOOS_TABLE_CAPTION'); ?>, <?php echo Text::_('JGLOBAL_SORTED_BY'); ?>
+					</caption>
 						<th scope="col" style="width:10%" class="d-none d-md-table-cell">
 						</th>
 						<th scope="col" style="width:1%">
```

## Test your Joomla component

1- install your component in Joomla version 4 to test it: Copy the files in the `administrator` folder into the `administrator` folder of your Joomla 4 installation.

2.the database has been changed, so it is necessary to update it. Open the `System | Information | Database` area as described in part 16. Select your component and click `Update Structure`.

![Joomla Published](/images/j4x16x1.png)

3. open the view of your component in the administration area and filter, sort and search for items in your component.

![Joomla Filter Sort and Search - Searchtools](/images/j4x20x1.png)
<img src="https://vg08.met.vgwort.de/na/5178de69ae0a4beeb708558127a43823" width="1" height="1" alt="">
