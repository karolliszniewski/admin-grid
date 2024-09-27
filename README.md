# 1. Register module

file `/Mageplaza/HelloWorld/view`

```php
<?php

use Magento\Framework\Component\ComponentRegistrar;

ComponentRegistrar::register(
    ComponentRegistrar::MODULE,
    
     // The name of the module we're registering
    'Mageplaza_HelloWorld',
    __DIR__
);
```

file `/Mageplaza/HelloWorld/etc/module.xml`

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Mageplaza_HelloWorld" setup_version="1.0.0"/>
</config>
```

# 2. create table in db

file `/Mageplaza/HelloWorld/Setup/InstallSchema.php`

```php
<?php

namespace Mageplaza\HelloWorld\Setup;

class InstallSchema implements \Magento\Framework\Setup\InstallSchemaInterface
{

	public function install(\Magento\Framework\Setup\SchemaSetupInterface $setup, \Magento\Framework\Setup\ModuleContextInterface $context)
	{
		$installer = $setup;
		$installer->startSetup();
		if (!$installer->tableExists('mageplaza_helloworld_post')) {
			$table = $installer->getConnection()->newTable(
				$installer->getTable('mageplaza_helloworld_post')
			)
				->addColumn(
					'post_id',
					\Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
					null,
					[
						'identity' => true,
						'nullable' => false,
						'primary'  => true,
						'unsigned' => true,
					],
					'Post ID'
				)
				->addColumn(
					'name',
					\Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
					255,
					['nullable => false'],
					'Post Name'
				)
				->addColumn(
					'url_key',
					\Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
					255,
					[],
					'Post URL Key'
				)
				->addColumn(
					'post_content',
					\Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
					'64k',
					[],
					'Post Post Content'
				)
				->addColumn(
					'tags',
					\Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
					255,
					[],
					'Post Tags'
				)
				->addColumn(
					'status',
					\Magento\Framework\DB\Ddl\Table::TYPE_INTEGER,
					1,
					[],
					'Post Status'
				)
				->addColumn(
					'featured_image',
					\Magento\Framework\DB\Ddl\Table::TYPE_TEXT,
					255,
					[],
					'Post Featured Image'
				)
				->addColumn(
					'created_at',
					\Magento\Framework\DB\Ddl\Table::TYPE_TIMESTAMP,
					null,
					['nullable' => false, 'default' => \Magento\Framework\DB\Ddl\Table::TIMESTAMP_INIT],
					'Created At'
				)->addColumn(
					'updated_at',
					\Magento\Framework\DB\Ddl\Table::TYPE_TIMESTAMP,
					null,
					['nullable' => false, 'default' => \Magento\Framework\DB\Ddl\Table::TIMESTAMP_INIT_UPDATE],
					'Updated At')
				->setComment('Post Table');
			$installer->getConnection()->createTable($table);

			$installer->getConnection()->addIndex(
				$installer->getTable('mageplaza_helloworld_post'),
				$setup->getIdxName(
					$installer->getTable('mageplaza_helloworld_post'),
					['name', 'url_key', 'post_content', 'tags', 'featured_image'],
					\Magento\Framework\DB\Adapter\AdapterInterface::INDEX_TYPE_FULLTEXT
				),
				['name', 'url_key', 'post_content', 'tags', 'featured_image'],
				\Magento\Framework\DB\Adapter\AdapterInterface::INDEX_TYPE_FULLTEXT
			);
		}
		$installer->endSetup();
	}
}
```

#3. Create Admin route

file `/Mageplaza/HelloWorld/etc/adminhtml/routes.xml`

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="admin">
        <route id="mageplaza_helloworld" frontName="mageplaza_helloworld">
            <module name="Mageplaza_HelloWorld"/>
        </route>
    </router>
</config>
```

# Create controller

file `Mageplaza/HelloWorld/Controller/Adminhtml/Post/Index.php`

```php
<?php

namespace Mageplaza\HelloWorld\Controller\Adminhtml\Post;

class Index extends \Magento\Backend\App\Action
{
	protected $resultPageFactory;

	public function __construct(
		\Magento\Backend\App\Action\Context $context,
		\Magento\Framework\View\Result\PageFactory $resultPageFactory
	)
	{
		parent::__construct($context);
		$this->resultPageFactory = $resultPageFactory;
	}

	public function execute()
	{
		$resultPage = $this->resultPageFactory->create();
		$resultPage->getConfig()->getTitle()->prepend((__('Posts')));

		return $resultPage;
	}


}
```

### - Create Admin Grid using Component

# 1. Declare resources

file: `/Mageplaza/HelloWorld/etc/di.xml`

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../../../../lib/internal/Magento/Framework/ObjectManager/etc/config.xsd">
    <type name="Magento\Framework\View\Element\UiComponent\DataProvider\CollectionFactory">
        <arguments>
            <argument name="collections" xsi:type="array">
                <item name="mageplaza_helloworld_post_listing_data_source" xsi:type="string">Mageplaza\HelloWorld\Model\ResourceModel\Post\Grid\Collection</item>
            </argument>
        </arguments>
    </type>
    <virtualType name="Mageplaza\HelloWorld\Model\ResourceModel\Post\Grid\Collection" type="Magento\Framework\View\Element\UiComponent\DataProvider\SearchResult">
        <arguments>
            <argument name="mainTable" xsi:type="string">mageplaza_helloworld_post</argument>
            <argument name="resourceModel" xsi:type="string">Mageplaza\HelloWorld\Model\ResourceModel\Post</argument>
        </arguments>
    </virtualType>
</config>
```

# 2. Create layout file
     
file `Mageplaza/HelloWorld/view/adminhtml/layout/mageplaza_helloworld_post_index.xml`

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../../../../../../lib/internal/Magento/Framework/View/Layout/etc/page_configuration.xsd">
    <update handle="styles"/>
    <body>
        <referenceContainer name="content">
            <uiComponent name="mageplaza_helloworld_post_listing"/>
        </referenceContainer>
    </body>
</page>
```

file `Megeplaza/Helloworld/view/adminhtml/ui_component/megeplaza_hellloworld_post_listing.xml`

```xml
<listing xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Ui:etc/ui_configuration.xsd">
    <argument name="data" xsi:type="array">
        <item name="js_config" xsi:type="array">
            <item name="provider" xsi:type="string">mageplaza_helloworld_post_listing.mageplaza_helloworld_post_listing_data_source</item>
            <item name="deps" xsi:type="string">mageplaza_helloworld_post_listing.mageplaza_helloworld_post_listing_data_source</item>
        </item>
        <item name="spinner" xsi:type="string">spinner_columns</item>
        <item name="buttons" xsi:type="array">
            <item name="add" xsi:type="array">
                <item name="name" xsi:type="string">add</item>
                <item name="label" xsi:type="string" translate="true">Add New Post</item>
                <item name="class" xsi:type="string">primary</item>
                <item name="url" xsi:type="string">*/*/new</item>
            </item>
        </item>
    </argument>
    <dataSource name="nameOfDataSource">
        <argument name="dataProvider" xsi:type="configurableObject">
            <argument name="class" xsi:type="string">Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider</argument>
            <argument name="name" xsi:type="string">mageplaza_helloworld_post_listing_data_source</argument>
            <argument name="primaryFieldName" xsi:type="string">post_id</argument>
            <argument name="requestFieldName" xsi:type="string">id</argument>
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="component" xsi:type="string">Magento_Ui/js/grid/provider</item>
                    <item name="update_url" xsi:type="url" path="mui/index/render"/>
                    <item name="storageConfig" xsi:type="array">
                        <item name="indexField" xsi:type="string">post_id</item>
                    </item>
                </item>
            </argument>
        </argument>
    </dataSource>
    <columns name="spinner_columns">
        <selectionsColumn name="ids">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="resizeEnabled" xsi:type="boolean">false</item>
                    <item name="resizeDefaultWidth" xsi:type="string">55</item>
                    <item name="indexField" xsi:type="string">post_id</item>
                </item>
            </argument>
        </selectionsColumn>
        <column name="post_id">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="filter" xsi:type="string">textRange</item>
                    <item name="sorting" xsi:type="string">asc</item>
                    <item name="label" xsi:type="string" translate="true">ID</item>
                </item>
            </argument>
        </column>
        <column name="name">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="filter" xsi:type="string">text</item>
                    <item name="editor" xsi:type="array">
                        <item name="editorType" xsi:type="string">text</item>
                        <item name="validation" xsi:type="array">
                            <item name="required-entry" xsi:type="boolean">true</item>
                        </item>
                    </item>
                    <item name="label" xsi:type="string" translate="true">Name</item>
                </item>
            </argument>
        </column>
        <column name="created_at" class="Magento\Ui\Component\Listing\Columns\Date">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="filter" xsi:type="string">dateRange</item>
                    <item name="component" xsi:type="string">Magento_Ui/js/grid/columns/date</item>
                    <item name="dataType" xsi:type="string">date</item>
                    <item name="label" xsi:type="string" translate="true">Created</item>
                </item>
            </argument>
        </column>
        <column name="updated_at" class="Magento\Ui\Component\Listing\Columns\Date">
            <argument name="data" xsi:type="array">
                <item name="config" xsi:type="array">
                    <item name="filter" xsi:type="string">dateRange</item>
                    <item name="component" xsi:type="string">Magento_Ui/js/grid/columns/date</item>
                    <item name="dataType" xsi:type="string">date</item>
                    <item name="label" xsi:type="string" translate="true">Modified</item>
                </item>
            </argument>
        </column>
    </columns>
</listing>
```

file `app/code/Mageplaza/HelloWorld/etc/adminhtml/menu.xml`

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Backend:etc/menu.xsd">
    <menu>
        <add id="Mageplaza_HelloWorld::helloworld" title="Hello World" module="Mageplaza_HelloWorld" sortOrder="51" resource="Mageplaza_HelloWorld::helloworld"/>
        <add id="Mageplaza_HelloWorld::post" title="Manage Posts" module="Mageplaza_HelloWorld" sortOrder="10" action="mageplaza_helloworld/post" resource="Mageplaza_HelloWorld::post" parent="Mageplaza_HelloWorld::helloworld"/>
        <add id="Mageplaza_HelloWorld::hello_configuration" title="Configuration" module="Mageplaza_HelloWorld" sortOrder="99" parent="Mageplaza_HelloWorld::helloworld" action="adminhtml/system_config/edit/section/helloworld" resource="Mageplaza_HelloWorld::helloworld_configuration"/>
    </menu>
</config>
```

file `app/code/Mageplaza/HelloWorld/Model/Post.php`


```php
<?php
namespace Mageplaza\HelloWorld\Model\ResourceModel;


class Post extends \Magento\Framework\Model\ResourceModel\Db\AbstractDb
{
	
	public function __construct(
		\Magento\Framework\Model\ResourceModel\Db\Context $context
	)
	{
		parent::__construct($context);
	}
	
	protected function _construct()
	{
		$this->_init('mageplaza_helloworld_post', 'post_id');
	}
	
}
```


file `app/code/Mageplaza/HelloWorld/Model/ResourceModel/Post/Collection.php`

```php
<?php
namespace Mageplaza\HelloWorld\Model\ResourceModel\Post;

class Collection extends \Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection
{
	protected $_idFieldName = 'post_id';
	protected $_eventPrefix = 'mageplaza_helloworld_post_collection';
	protected $_eventObject = 'post_collection';

	/**
	 * Define resource model
	 *
	 * @return void
	 */
	protected function _construct()
	{
		$this->_init('Mageplaza\HelloWorld\Model\Post', 'Mageplaza\HelloWorld\Model\ResourceModel\Post');
	}

}
```

file `app/code/Mageplaza/HelloWorld/view/mageplaza_helloworld_post_index.xml`

```xml
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../../../../../../../lib/internal/Magento/Framework/View/Layout/etc/page_configuration.xsd">
    <update handle="styles"/>
    <body>
        <referenceContainer name="content">
            <uiComponent name="mageplaza_helloworld_post_listing"/>
        </referenceContainer>
    </body>
</page>
```


![image](https://github.com/user-attachments/assets/85fd8250-b2fd-408e-a473-bdf43f69f81c)







