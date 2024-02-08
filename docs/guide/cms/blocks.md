# Building Blocks

Blocks are elements used in the CMS module to display and configure data. Blocks are dropped into the placeholders of a [CMS Layout](/guide/cms/cmslayouts). An example of a block could be a paragraph tag where the user can add the content. LUYA CMS module is shipped with some default blocks, but you can create your own elements.

![Block Admin](../img/block-admin.png "Block Admin")

## Create a new block

::: tip
Use `./vendor/bin/luya cms/block/create` console command to generate a block.
:::

You can add blocks to your application or to a module. In either case, the folder where the blocks are stored must be named as **blocks**. Additionally blocks should have the suffix `Block` in their filename.

For example, we create a block `TextTransformBlock` and store it in `app/blocks` or `app/modules/yourmodule/blocks`.

This is how the `TextTransformBlock` could look like in your code:

```php
<?php
namespace app\blocks;

class TextTransformBlock extends \luya\cms\base\PhpBlock
{
    public function icon()
    {
        return 'extension'; // https://material.io/resources/icons/
    }
    
    public function name()
    {
        return 'Transformed Text';
    }
    
    public function config()
    {
        return [
            'vars' => [
                ['var' => 'mytext', 'label' => 'The Text', 'type' => self::TYPE_TEXT],
            ],
        ];
    }

    public function extraVars()
    {
        return [
            'textTransformed' => strtoupper($this->getVarValue('mytext')),
        ];
    }

    public function admin()
    {
        return 'Administrations View: ` vars.mytext `';
    }
}
```

Define help information for the admin view:

```php
public function getFieldHelp()
{
    return [
        'content' => 'An explain example of what this var does and where it is displayed.',
    ];
}
```

::: tip Block Types
Read more about details regarding <class name="luya\cms\base\InternalBaseBlock" method="config" /> in [Block Types](blocktypes)
:::

As we have switched to <class name="luya\cms\base\PhpBlock" /> by default you now have to create also a view file, which is located in the view folder of your application: `app/blocks/views`. The view itself must have the same name as the class name of your block, e.g. `TextTransformBlock.php`. 

In the example above, the view file should look like this:

```php
<?php
/* @var $this \luya\cms\base\PhpBlockView */
?>

<?php if ($this->varValue('mytext')): ?>
    <h1><?= $this->extraValue('textTransformed'); ?></h1>
<?php endif; ?>
```

In order to retrieve values from configurations (`$this->[METHOD]`):

|Type|Method
|----|------
|var|<class name="luya\cms\base\PhpBlockView" method="varValue" />
|cfg|<class name="luya\cms\base\PhpBlockView" method="cfgValue" />
|extra|<class name="luya\cms\base\PhpBlockView" method="extraValue" />
|placeholder|<class name="luya\cms\base\PhpBlockView" method="placeholderValue" />

Check the <class name="luya\cms\base\PhpBlockView" /> for full method reference to use inside the PHP block view.

### Block rendering internals

For a more detailed understanding of the nexus of a block and its view file, the corresponding function call paths are listed below.

In admin context:

`NavItemPage::getPlaceholder()`
&rarr; `NavItemPage::getBlockItem()`
&rarr; `BlockInterface::renderAdmin()`
&rarr; `PhpBlockInterface::admin()` implemented in your block, e.g. `TextTransformBlock::admin()` in `app/blocks/TextTransformBlock.php`

In frontend context:

`NavItemPage::renderPlaceholder()`
&rarr; `NavItemPage::renderPlaceholderRecursive()`
&rarr; `BlockInterface::renderFrontend()`
&rarr; `PhpBlockInterface::frontend()`
&rarr; `View::render()`
&rarr; `View::renderFile()`
&rarr; block view file, e.g. `app/blocks/views/TextTransformBlock.php`

## Register and import

After creating a block, you have to *import* it into your application. The reason behind the import process is to avoid rely on database structure (which cant be tracked in a VCS or SVN easily) and to work with PHP files you can manage in version control system (e.g. GitHub or BitBucket). Run the [Import Command](/guide/app/console):

```sh
./vendor/bin/luya import
```

This will add or update the block into the CMS system. If you rename or remove a Block from your application, the old block will be deleted from your database.

### Module blocks

When you add a block inside of a module you have to define the <class name="luya\cms\base\PhpBlock" prop="module" /> properties, this will ensure the view file will be found in the correct folder. This way, you can redistributed blocks with your own package to other users:

```php
class TestBlock extends \luya\cms\base\PhpBlock
{
    public $module = 'mymodule';
}
```

## Caching

To speed up your system you can enable the cache for each block by the definition of a [caching component](https://www.yiiframework.com/doc-2.0/guide-caching-data.html#cache-components) in your configs. Block caching is disabled by default for all blocks <class name="luya\cms\base\PhpBlock" prop="cacheEnabled" />

```php
class MyTestBlock extends \luya\cms\base\PhpBlock
{
    public $cacheEnabled = true;
}
```

This will cache the block for 60 minutes but you can adjust the number of seconds the block should be cached by defining the <class name="luya\cms\base\PhpBlock" prop="cacheExpiration" /> property:

```php
public $cacheExpiration = 60;
```

You can enable block caching for a block event if the caching component is not registered, so you can redistribute blocks and the behavior of them.

## Env / Context Information

Each block is placed in an environment (env). You can access those information inside your block with <class name="luya\cms\base\PhpBlock" method="getEnvOption" /> logic:

```php
$this->getEnvOption($key, $defaultValue);
```

The following keys are available:

+ **id**: Returns the unique identifier from the CMS context.
+ **blockId**: Returns the id of this block (unique identifier).
+ **context**: Returns `'admin'` or `'frontend'` to find out in which context you are.
+ **pageObject**: Returns the <class name="luya\cms\models\NavItemPage" /> object where the block is located.
+ **isFirst**: Returns whether this block is the first in its placeholder or not.
+ **isLast**: Return whether this block is the last in its placeholder or not.
+ **index**: Returns the index number/position within this placeholder.
+ **itemsCount**: Returns the number of items inside this placeholder.
+ **isPrevEqual**: Returns whether the previous item is of the same origin (block type, like text block) as the current.
+ **isNextEqual**: Returns whether the next item is of the same origin (block type, like text block) as the current.
+ **equalIndex**: Returns the current index number/position of this element within the list of *same* elements.

Thereof you can also retrieve the related <class name="luya\cms\models\NavItem" /> and <class name="luya\cms\models\Nav" /> objects
via <class name="luya\cms\models\NavItemPage" method="getNavItem" /> and <class name="luya\cms\models\NavItem" method="getNav" /> as follows:

```php
$this->getEnvOption('pageObject')->navItem->nav;
```

Inside a PHP block view call instead:

```php
$this->block->getEnvOption($key, $defaultValue);
```

Or you can easily access these properties through convenient shortcuts, e.g. `$this->isPrevEqual`.

For example, they are useful to make a container layout block which auto close/open the row when working with a grid system like the one from Bootstrap:

```php
<?php if (!$this->isPrevEqual): ?>
<div class="row">
<?php endif; ?>

    <div class="col-md-3">
       <h1>The Block Content.</h1>
    </div>

<?php if (!$this->isNextEqual): ?>
</div>
<?php endif; ?>
```

The above example would only open the row element once and closes the row container when the next element is not an element of the current block.

#### Properties from CMS page

If there are any CMS properties defined you can access them like this:

```php
$this->getEnvOption('pageObject')->navItem->nav->getProperty('myCustomProperty');
```

If there is a property defined you will get the property object otherwise returning `false`.

## Register Assets and JavaScript/CSS

Sometimes your block should also register some CSS or JavaScript files, therefore you can access the global <class name="luya\web\View" /> object inside of your PHP view template. It is quite similar to registering other assets with the difference that you are accessing the global scope view instead of the view on `$this`.

Assuming the below code is the PHP view of your block:

```php
MyBlockAsset::register($this);
```

Now the [Asset](../app/assets) is registered in the application view object.

It's also possible to just register certain JavaScript or CSS code, therefore use `registerJs()` or `registerCss()` directly in the view files of the block.

Registering some JavaScript code:

```php
$this->registerJs("
    console.log('hello world');
");
```

Or register a variable:

```php
$this->registerJsVar("foo", "bar"); // var foo = 'bar'
```

And for CSS inline styles use:

```php
$this->registerCss("
    .data { color: red; }
"); 
```

## AJAX Requests in Block

To implement AJAX inside a block the following concept is used:

+ <class name="luya\cms\base\InternalBaseBlock" method="createAjaxLink" />: Create the link to the callback, this URL must be used for your AJAX requests.
+ `callback...()`: Define a callback, you have to prefix the method with *callback*.

Create a callback and define all parameters. The callback is what the URL returns to your JavaScript which can be HTML or JSON.

```php
public function callbackHelloWorld($time)
{
    return 'hallo world ' . $time;
}
```

The above callback requires the parameter `$time` and must be called trough an AJAX call inside of the JavaScript, to create the URL for this specific callback we are going to use `createAjaxLink()`:

```php
$this->createAjaxLink('HellWorld', ['time' => time()]);
```

You could store this created link from above inside your extras vars and pass it to the JavaScript.

You can pass additional values to the callback by using the post AJAX method and collect them in your callback via `Yii::$app->request->post()`. The get parameters are used to resolve the callback.


## Block groups

There is the ability to manage the block groups via classes. 
You can add new `blockgroups` to your application on which your block depends, when you run the import command LUYA will add/update the blocks into the provided groups.

To add new block groups create folder in your `@app` namespace, or inside a module with the name `blockgroups`, to add a folder create class like this `app\blockgroups\MySuperGroup`:

```php
<?php
namespace app\blockgroups;

class MySuperGroup extends \luya\cms\base\BlockGroup
{
    public function identifier()
    {
        return 'my-super-group';
    }
    
    public function label()
    {
        return 'My Super Group Elements';
    }
    
    public function getPosition()
    {
        return 30;
    }
}
```

> The position of the block will start from lower to higher, means 1 will be at the top of the groups list in the administration and even higher will be more at the bottom.

The folder will be created on import. Now blocks can belong to this folder, to do so override the <class name="luya\cms\base\BlockInterface" method="blockGroup"/> method of your block:

```php
public function blockGroup()
{
    return \app\blockgroups\MySuperGroup::className();
}
```

You can also use one of the predefined group block class:

+ `\luya\cms\frontend\blockgroups\MainGroup` (this is default group for all blocks)
+ `\luya\cms\frontend\blockgroups\LayoutGroup`
+ `\luya\cms\frontend\blockgroups\ProjectGroup`
+ `\luya\cms\frontend\blockgroups\DevelopmentGroup`
+ `\luya\cms\frontend\blockgroups\MediaGroup`
+ `\luya\cms\frontend\blockgroups\TextGroup`

## Block variations

In order to extend a block with a special behaviour the <class name="luya\cms\admin\Module" prop="blockVariations" /> property of the `cmsadmin` module can be configured inside your general config like in the example below:

```php
'cmsadmin' => [
    'class' => 'luya\cms\admin\Module',
    'blockVariations' => [
        TextBlock::variations()
            ->add('bold-flavor', 'Bold font with Markdown')->cfgs(['cssClass' => 'bold-font'])->vars(['textType' => 1])
            ->add('italic-flavor', 'Italic Font')->cfgs(['cssClass' => 'italic-font'])
            ->register(),
    ]
],
```

Maybe the application block is not known inside the config files, then there is an alternative option to define block variations:

```php
'cmsadmin' => [
    'class' => 'luya\cms\admin\Module',
    'blockVariations' => [
         'app\blocks\TextBlock' => [
             'variation1' => [
                 'title' => 'Super Bold Text',
                 'vars' => ['cssClass' => 'bold-font-css-class'],
                 'cfgs' => [], // will be ignore as it's empty, so you can also just remove this part.
                 'extras' => [], // will be ignore as it's empty, so you can also just remove this part.
                 'is_default' => 0,
             ],
         ]
    ]
]
```

## Block preview

To give the user a better understanding how you block could looks like you can store a JPEG image under `app/images/blocks` or `app/modules/yourmodule/images/blocks` with same name as the block e.g. `TextTransformBlock.jpg`.
This image will be display on hover over a block name in the navigation. Make sure that this file size is so small as possible (maybe use a image optimizer) and image size is under 400x400 pixel.
