# Zertifizierung Zusammenfassung
---


## High-level Magento architecture
---

### Describe Magento Code pools

Magento codepools stand for three different locations, where magento looks for extensions. We have the following three ones: core, communty, local and Magento starts looking for modules in local, than it tries to find classes in community and after that, the core section is searched for neccesary classes.

- app/code/core
	Here we find all Magento Modules, that come with a fresh installations. This codepool resembles the
	core of Magento and is **never** to be touched or modified!
	
- app/code/community
	This is the place for third-party modules or self developed extensions. It works exactly like /core, except 
	for the point, that magento loads everything in communty first
	
- app/code/local
	This is the place, where magento tries to load class-files from first. For example, here one could override
	classes from core in one way. There is another, which we will describe later on. So, if Magento finds its
	modified core-file in /local first, the original file from /core is not loaded anymore.
	

### <a id="module_setup"></a>Describe the typical Magento module structure

There are a lot of modules out there right now, and all certainly will look similar to this:
First, there is the *module activation file*, which is located in app/etc/modules and is a simple XML-File, that contains the Module name, its namespace and code-pool and as well the "is it active"-Flag as also dependencies 
from other modules, if needed.
It basically looks like this:


	<?xml version="1.0" ?>

	<config>
    	<modules>
        	<Cert_Exercise1>
            	<active>true</active>
            	<codePool>community</codePool>
	        </Cert_Exercise1>
   		</modules>
	</config>
	
Then we need to have a basic configuration file in app/code/codepool/Namespace/Modulname/etc/ which is to be named config.xml

This, in its simplest form, contains the module name, namespace and version of the module, so Magento sees, if it has to load any update or install-scripts. In this file we declare all other files, our module needs. For example controllers, routes, layout-updates, Block-, Helper- and Modelclasses.

Furthermore our module can have those folders containing the classes:

- /Block
- /controllers (notice the small "c")
- /Helper
- /Model
- /sql

### Describe the location of layout and template files

Layout and template files, in before Magento2 ;), do not come within our modules folder. We use the folders located in app/design/frontend or app/design/backend - here we have our design areas, containing our design packages. The **base**-package in each area is not to be touched, because its the default fallback of magento. If a file is at last not be found there, we have a problem.
So we use the default package, to set our theme (or, if we have a more complex theme/design, we can create our own package there). Within the themes folder, we have

- /locale
- /layout
- /template

as our main folders. Within the template folder, we keep our .phtml-Template files which contain dynamic HTML-Stuff to render our design. Here we define the contentual Blocks of our theme. In the layout folder, we define the structural blocking of our pages in XML. Those structure blocks will be filled with content from the template files.

### Describe the location of Magento skin and JS files

We keep skin and js files in /magentoroot/skin or /magentoroot/js for general purposes, and we kann put them within our design folders, if we just need them for one specific design.

### Identify and explain the main Magento design areas (adminhtml and frontend)

The main design areas are just frontend and adminhtml. We have a different set of layout and templates used for frontend design and functionality as well as for our backend. To not mix them up, they are splittet. In each of app/design, /skin and /js, we have as first folders "frontend" and "adminhtml", which is to keep the files seperated. Also Magento uses this, to load the correct files via the autoloader for any given area. That means, we can have a product/list.phtml for frontend and another file, named product/list.phtml for the backend and Magento will not mix them up.

### Explain class naming conventions and their relationship with the autoloader

Class naming convention is a real simple thing in Magento. Classes are named after there physical location in relation to your magentoroot, where they can be found.
For example `Mage_Sales_Model_Order` is found in app/code/core/**Mage/Sales/Model/Order.php**
If PHP is not able to find a class, it just passes the classname to Magentos autoloader and it will do the magic needed to find the class. The Magento autoloader will convert the classname at underscores "_" to whitespaces " " and will then replace the whitespaces with directoryseperators. This is to be independent from "/" and "\" on Unix/Linux/Windows machines. After that is done, it simply puts an ".php" to the end and voíla, there is your class Magento!

### Describe methods for resolving module conflicts

First we have to find out the improper module. Then we need to see, if there are class rewrites, that do not work. If we have found any, we have to check if we can integrate one class into the other, without destroying the functionaliy. For example, if both modules override different functions, we can change the inheritance order to prevent Magento from trying to load both classes at the same time.
We also could try to modify the improper module in a way, that we can see the timings of calls or the execution order of it, compared to magento core, to find any disturbances in the force, err in Magento.

## Magento configuration
---

### <a id="load_config"></a>Explain how Magento loads and manipulates configuration information

If magento boots up the first time, it checks the app/etc/config.xml for initial information on version, database connection, installed-flag and localization. After that, it writes the app/etc/local.xml for database account and adminhtml path in frontend. 
After that it will go to app/etc/modules and read all files there, which end to .xml. Done that, magento will go to all module folders, which are specified in app/etc/modules` .xml-files and load the respective config.xml files.
At this point Magento knows of all modules from local/community/core it has to load. But to prevent bad modules to do bad things to our crucial information on databases and other sensible data, it loads the local.xml again to override all malicious setups with at least one functioning and correct set of information

### Describe class group configuration and use in factory methods

Magento groups its classes into 2 different factory-groups.

- Models
- Helpers

With the appropiate `Mage::getModel()` or `Mage::Helper()` we start the factoring process. We pass a string to these functions which looks like `catalog/product`. That means, Magento is going to search its configuration for any `<catalog>` node and sees the class prefix in its `<class>` node. In this case it would be `Mage_Catalog_Model` (if we use `Mage::getModel('catalog/product)`. While having the prefix of our desired class, Magento uses the part after the "/" to find the fitting model .php-file in the "Model" folder of the "Catalog" Extension. So `catalog/product` will get parsed to `Mage_Catalog_Model`/product and this will get convert into `Mage_Catalog_Model_Product` and finally the autoloader puts its famous ".php" at the end and we have our Model class.
The exact same thing happens, if we use the `Mage::Helper()` method. This time, Magento searches all defined Helpers (from all config.xml files) to get the correct helper class. Though Magento uses the default Helper class `Data.php` we do not have to specify this unless we want a more specified helper.

### Describe the process and configuration of class overrides in Magento

To override any given class in Magento in our own module, we simply define a `<rewrite>` node in our config.xml. If Magento tries to load our previously mentioned `catalog/product`, it will not only search for all `<catalog>` nodes, but especially for any `<rewrite>` nodes, which may be found in any `<catalog>` nodes. These rewrites define the class prefix of a class, that is to be used instead of the default class prefix defined in the original `<catalog>` node.
**This is the Magento-Way**

There is, though not recommended, another way to override classes. As explained earlier, Magento looks in the app/code/local folder first. If we simply put a class into a folder like app/code/local/Mage/Catalog/Model/Product.php magento will use this file everytime, we want to autoload our `catalog/product`. We do not need to specify this override in any way, it just works. But its extremy difficult to debug core modules if someone just overrides the original class this way, because, as said, there is no specification to be made to look for.

### Register an observer

Observers in Magento are hooked to so called events. Events are fired every now and then in Magento to signalize that certain things have happen. With observers, we can hook onto that place and execute our own code to make more stuff happen at that time.
To do that, we need to specify an `<events>` node in our config.xml within the `<global>` node, name the event that we want to listen to, specify that we want to observe this event with an `<observers>` node, name our observer and specify the class and method that contains the code we want to execute, when the event occurs.
This can look like this:

	<events>
        <controller_action_predispatch>
            <observers>
                <cert_exercise1_observer>
                    <class>Cert_Exercise1_Model_Observer</class>
                    <method>homeRedirect</method>
                </cert_exercise1_observer>
            </observers>
        </controller_action_predispatch>
    </events>


This one will execute the `homeRedirect()` method from our observer located at app/code/community/Cert/Exercise1/Model/Observer.php.

### Identify the function and proper use of automatically available events, including *_load_after, etc.

Thoug events are specified, there are a lot of auto generated events in Magento. Especially the _predispatch and _postdispatch events that are fired from every controller within Magento.
The above mentioned *_load_after and *_load_before events are fired, if Magento loads any Model. With observers registered, for example on `<catalog_product_load_before>` we can modify the product model before its completely loaded. We could add more attributes to load, we even can exchange the whole model to a different model in that case.

### Set up a cron job

Magento is able to uses the cron program on the machine it is installed to to automate certain tasks, like log cleaning, sitemap generation or sending mass newsletters.
A cronjob setup in Magento is pretty simple: It is, like always, just another node in our config.xml. This one can look similar to this one (which is from the sitemap module):

     <crontab>
        <jobs>
            <sitemap_generate>
                <run>
                    <model>sitemap/observer::scheduledGenerateSitemaps</model>
                </run>
            </sitemap_generate>
        </jobs>
    </crontab>

This one executes the `scheduledGenerateSitemaps()` method of the `Sitemap/Observer.php` everytime we specified in the Magento backend. We could also add a `<schedule>` node which contains a `<cron_expr>` node which contains a standard cron expression. This will work as the standard systems crontab.


## Internationalization
---

### Describe how to plan for internationalization of a Magento site

If we need to implement our shop with different languages, it is useful to setup different StoreViews for each language, we want to use. That way, we can easily seperate different settings and localizations for our different shops. Magento as well can easier get the correct files for each shop.

### Describe the use of Magento translate classes and translate files

Magento brings a whole factory for translating texts into different languages as well as different locations for translation files.

- app/locale/[locale]/ for module translation
- app/design/[package]/[theme]/translate.csv for theme translation
- `core_translate` table in the database for basic translation and fallback reasons

While the above order describes the order of loading translations, the order of getting them into our template is the oppite around. Because the Database loads last, its information is the one first used on getting. Thats why everything, thats in there overrides any other translation.

A small example:

We can use the themes translate.csv to manually override any modules translation, by putting the source string into the translate.csv and adding a translation.

### Describe the advantages and disadvantages of using subdomains and subdirectories in internationalization

I do not remember. Please help!


## Application initialization
---

### Describe the steps for application initialization

To start Magento up, first the whole configuration is loaded, as described earlier in this document. After having a functional configuration, Magento instantiates the `Mage` class and then runs the `run()` or the `app()` methods. In both cases Magento start to evaluate the request, which is passed on to the `index.php` file in our Magento root directory.
The requestpath is seperated and passed into the front controller `Mage_Core_Controller_Varien_Action` which gathers all data requestet, builds the layout XML structure and renders it to HTML.

### Describe the role of the system entry point, index.php

The first thing a request will encounter is the `index.php` file in our Magento root directory. In here, basic checks are run, on which Magento can decide to run, or not to run.
Things that get checked here:

- PHP version >= 5.2.0
- Error report level
- Compiling enabled?
- Maintenance flag set?
- MAGE_DEVELOPER_MODE
- Run type / Run code

In the last line `Mage::run($mageRunCode, $mageRunType);` is executed and Magento starts up.
Basically, the `index.php` file is the root and backbone of a Magento installation. Without it, Magento won´t run at all.

### Describe the role of the front controller

The `Mage_Core_Controller_Varien_Action` is the first point of evaluating a request. In here, we dig deep into Magentos classes to fulfill our request. We dispatch events, to let everyone know, that controllers have been executed, or layout has been generated.


### <a id="front_controller_events"></a>Identify uses for events fired in the front controller

The most important events fired in the front controller are the post- and predispatch events. That basically are events, fired before or after the event is fired.

Sounds confusing, but it is very useful. As default we get three events for each function.

- `controller_action_predispatch` - Which is fired everytime any controller will be loaded.
- `controller_action_predispatch_'.$this->getRequest()->getRouteName()` - Which is fired for specific frontend routes - for example everytime before a customer invokes an action this event will be transformed to `controller_action_predispatch_customer_account`
- `controller_action_predispatch_'.$this->getFullActionName()` - Which is fired everytime if a specific frontend action is executed. To stick to my example from above, we could get a `controller_action_predispatch_customer_account_login` event everytime before a customer logs in.

---

- `controller_action_postdispatch`
- `controller_action_postdispatch_'.$this->getRequest()->getRouteName()`
- `controller_action_postdispatch_'.$this->getFullActionName()`

The last three events are exactly the same as the first ones, but those are fired just AFTER the respective things have happend.

### <a id="url_structure"></a>Describe URL structure/processing in Magento

Every URL in Magento is translated to a module/controller/action pattern. Customer/account/login would refer to the `loginAction()` method within the `AccountController.php` file in the customer module of Magento.

### Describe the URL rewrite process

Within our `config.xml` file, we are able to rewrite certain URLs to any other controller that exists in our shop. Because controllers in Magento are not rewriteable, you simple redirect the URL which launches a specific controller to another URL, which then launches any controller desired.
Such magic would look like the following:

	<global>
	        <rewrite>
	            <googlecheckout>
	                <core>
	                <from>/^.*?googlecheckout\/api/</from>
	                <to>googlecheckout/api</to>
	            </googlecheckout>
	        </rewrite>
	</global>

This example taken from the GoogleCheckout module of Magento should describe the rewriting pretty well. We have a `<rewrite>` node and instead of substituting a class, we substitute an URL. The important part are the `<from>` and `<to>` nodes, which are the actual URL-Rewriting.

### Describe request routing/request flow in Magento

Whenever a request hits the webserver, the host is cut away and the rest is used to start the corresponding module/controller/action flow. This is already described above in [Describe URL structure/processing in Magento](#url_structure).

### Describe how Magento determines which controller to use and how to customize route-to-controller resolution

Whitin the default router model of Magento, the request URL is taken apart to determine Module, Controller and Action to use. This happens in `Mage_Core_Controller_Varien_Action` and its `_forward()` method. Here the action name is set and the request is forwared to the `dispatch()` method, which tries to call the actionMethod in the request. If this is not possible, the noroute-Action is called, and if this is not possible either, an Exception is thrown.

### Describe the steps needed to create and register a new module

Registering a new module in Magento is pretty easy. We effectively just need two files.

- a Module activation file in app/etc/modules/Namespace_Modulname.xml
- a Module configuration file in app/code/codePool/Namepsace/Modulname/etc/config.xml

This is also described [here](#module_setup)


### Describe the effect of module dependencies

Sometimes we need to have other modules installed to run our own module. To ensure, this is the case, we can use a `<depends>` node in our module activation file.

This can look like this:

	<config>
   		 <modules>
   	    	 <Mage_Api>
   	        	 <active>true</active>
   	         	<codePool>core</codePool>
   	         	<depends>
   	            	 <Mage_Core />
   	        	</depends>
			</Mage_Api>
		</modules>
	</config>

If the config loading process finds one of these `<depends>`nodes, it will only register this module, if there is an `<active>true</active>` node in the node, that is named in the dependency.

In this case, the API module will only load if the `Mage_Core` module is installed and enabled.

### Describe different types of configuration files and their load priorities

This is already described [here](#load_config).

### Identify the steps in the request flow in which:
… … … … … … … … … … 

#### Design Data is populated

Withing the execution of a controller, Magento starts to gather neccessary information and data via models and ressource models, passes it through to Blocks and brings it into the template, where the information is rendered into HTML.

#### Layout configuration files are parsed

While rendering the HTML-Output of a block, the layout is loaded. Each structural block in the layout will get its name and alias, so it can be referred to, after gathering all data. These files are parsed before actual content will be rendered.

#### Layout is compiled


#### Output is rendered

Output is only rendered in so called output blocks. These blocks have an attribute "_toHtml" which means, they are directly involved in the response which is sent to the browser. These blocks have the responsibility to render their child blocks and therefore to render the whole page. This is normally done in the last part of the generation of the response.


### <a id="rendering"></a>Describe how and when Magento renders content to the browser

With the `toHtml()` or the `$this->getChildHtml()` in Blocks and Templates, Magento starts to render all in layout defined .phtml files, that are in the child chain of the root block. The root block is defined in the `page.xml` file in each Magento´s base/default theme.

### Describe how and when Magento flushes output variables using the front controller

Help please!

## Rendering
---

### Define and describe the use of themes in Magento

Themes in Magento represent the specific design of one specific shop/store view. Each storeview in Magento can only use one theme at a time.

### Define and describe the use of design packages

Design packages contain one or more themes, which is just for structuring your theme world in your shop.
As an Example of the last two topics, this should do fine:

We have a Magento store that sells Jackets, Shoes and computer hardware at the same time. While we do not want to have them look all the time, we create two design packages:

- Clothing
- Technical

This way, we have separated our shops designwise into two different areas. Now we create themes and assign them eventually as follows:

- Clothing
	- Jacket shop in summer design
	- Jacket shop in winter design
	- Shoe shop in summer design
	- Shoe shop in winter design
- Technical
	- Computer hardware
	- Home entertainment
	
As you can see, I have created more designs for specific reasons. The theme/package structure of Magento lets us easily pack themes together for the same purpose. As well as its perfectly easy to to small changes in a theme.
We simply create a new theme in our package and make only the neccessary changes. As long as we have another design (that is thematically inherited by our new one), Magento automatically falls back into this theme, if there are templates missing.

### Describe the process of defining template file paths

Template files are stored in our `app/design/` location and are divided in a structure like this:

- frontend/base/default/template - layout - locale
- frontend/default/default …
- adminhtml …

This way we exactly know where to look for our files. The `base/default` path represents the ultimate fallback destination and should **never** be touched. We can create more designs in the base-package, but that is also not recommended. It is better to put them into the default-package, since the `default/default` design is the second ultimate fallback and the default fallback in Magento.

### Describe the programmatic structure of blocks

Blocks in Magento gather the data provided by the models and put them in a way, we can use in our templates. So, for example, if we want to display customer information in a template, we create a block, which loads the `customer/customer` model, gathers the data needed and sets it as template variables. This way we can use these variables in our template and display them.
The block also is able to make changes to its layout or assigned template-files. This works in the same manner, as we can do this via layout XML.

### Describe the relationship between templates and blocks

Since a template is somewhat inherited by a block, they work very close together. In fact, the `Mage_Core_Block_Template` includes .phtml files directly, which means the templates HTLM and PHP directions appear within the blocks instance. That way we can use variables passed into the block via every child of it.

### Describe the stages in the lifecycle of a block

First a block gets instanciated by a controller or the layouts `_toHtml()` method. Second he gets his data through various models, which are instanciated by controllers and gathers its data. Then he renders his template and puts it into the HTTP-Response. After that, his life is over and its instance is never beeing used anymore in this request.

### Describe events fired in blocks

Basically we have 4 different, automatically fired events which work mostly the same as the [front controller events](#front_controller_events). These events are:

- `core_block_abstract_prepare_layout_before`
- `core_block_abstract_prepare_layout_after`
- `core_block_abstract_to_html_before`
- `core_block_abstract_to_html_after`

After reading about the front controller events, no more explanation should be neccessary.

### Identify different types of blocks

There are `Mage_Core_Block_Template` blocks, which are used to render template files into HTML. Then we have `Mage_Core_Block_Text` which only displays test. Besides that, we have more specialized blocks. For example the `Mage_Catalog_Block_Product_List` which is capable of displaying a list including several `Mage_Catalog_Block_Product` blocks.

### Describe block instantiation

We can use `createBlock('type/of_block)` of the `Mage_Core_Model_Layout` to get ourselves an instance of a block. This method is no factory method like `Mage::getModel()` or `Mage::helper()`. This will parse the name and return an instance of the desired block.

### Explain different mechanisms for disabling block output

We could remve the template of the block, by using `$block->setTemplate('');`, so the block has nothing to display. A rather hard way (and not recommended also) is to overide the page.xml file in `base/default/layout` and disable the root output by removing the `output="toHtml"` attribute of it.

### Describe how a typical block is rendered

This is described [here](#rendering)

### <a id="page_code"></a>Describe the elements of Magento's layout XML schema, including the major layout directives

A basic layout.xml could look like this:

	<?xml version="1.0" ?>
	<layout>

    	<catalog_product_view>
        	<reference name="content">
            	<block type="exercise3/comment" template="exercise3.phtml" />
	        </reference>
    	</catalog_product_view>

	    <catalog_category_default>
    	    <reference name="content">
        	    <block type="exercise3/category_comment" after="-" template="exercise3_cat.phtml" />
	        </reference>
    	</catalog_category_default>
    
	</layout>

Within this file, we declare that this part of the XML is `<layout>` and that we have to handles, on which we want layout updates to happen.
Everytime the `<catalog_product_view>` handle is rendered, we access the content block with a `<reference name="content">` node and tell Magento to insert another `<block type="exercise3/comment">` into the said content block.

Instead of using specific handles like above, we simply could use the `<default>` handle, to have our layout updated on every page in Magento.
In addition to this, we have more structues, like the following:

Removal of blocks:

	<reference name="blockname">
   		<remove name="block_to_be_removed_name"/>
	</reference>
	

Insertion of js/css

    <default>
        <reference name="head">
            <action method="addCss"><stylesheet>css/css.file</stylesheet></action>
        </reference>
    </default>

and

    <catalog_product_view>
        <reference name="head">
            <action method="addItem">
                <type>skin_js</type>
                <name>js/js.file</name>
            </action>
        </reference>
    </catalog_product_view>
    
With layout XML we nearly can do anything. :)

### Register layout XML files

To use our layout XML files, we have to tell Magento about these layout updates via the `config.xml`. We create a `<frontend>` that contains a `<layout>`, the obvious `<updates>` node, our own layout update handle `<Namespace_Modulname>` and a `<file>` node, which contains in plain text the path to our layout XML file relative the the `base/default` path.

To put it in code:

    <frontend>
        <layout>
            <updates>
                <Namespace_Modulname>
                    <file>layoutxml.file</file>
                </Namespace_Modulname>
            </updates>
        </layout>
    </frontend>
    
### Create and add code to pages

See [here](#page_code) - there we add JS and CSS code to our page.

### Explain how variables can be passed to block instances via layout XML

For this, we can use the `<action method="set<variablename>">values_here</action>` tags within a `<reference>` node.

### Describe various ways to add and customize JavaScript to specific request scopes

Also described [here](#page_codes) - but bare in mind to use layouthandles to specify on which page you want your code to be.

### Create frontend widgets and describe widget architecture

To create a widget, we need three things at least:

- a widget.xml within the /etc folder of our module
- a `Data.php` Helper in our Module
- a Block which extends `Mage_Core_Block_Template` and implements `Mage_Widget_Block_Interface`

The helper can be empty, but we need the appropiate class in our module to make the widget work. Within the block, we can gather data or prepare it to show within our widget.
The important part of the widget is the `widget.xml`, which contains the structure of our widget. We declare type, name, title and parameters of a widget like this:

	<?xml version="1.0"?>
	<widgets>
    	<exercise2 type="exercise2/widget_ercercise2">
       	 	<name>Exercise2 Widget</name>
        	<description>Widget from Certification Stuff</description>
        	<parameters>
            	<title translate="label">
                	<visible>1</visible>
                	<label>Exercise2</label>
                	<type>image</type>
            	</title>
        	</parameters>
    	</exercise2>
	</widgets>
	
Most important here, besides the self explaining things, is the `<parameters>` node, in which we declare the paramters our widget expects. (wow.)
Important here is the `<visible>` node and the `<type>` node, which define if the parameter field is visible when selected and what type it is. There are variants like text, textfield, image or file.

Widgets are basically normal template blocks, which can be used in the WYSIWYG-Editor in CMS-Blocks and sites. They are some sort of interactive blocks, where the user is able to define the content manually and not programmatically like in all other blocks Magento has to offer.

## ORM and Database
---

### Describe the basic concepts of models, resource models, and collections, and the relationship they have to one another

Well, lets start with the models. Models in Magento represent the data organisation. A model holds and gathers all data from data sources, which are needed. There are models for customers, orders and adresses, for example. A model provides the information it gathers to other models and instances, which rely on its data and which need to further evaluate the data. These other models and instances refer to the MVC (Model-View-Controller) concept Magento is based on. In this, the models represent the pure data side.
But Magento went further than just MVC - Magento has resource models and collections as well.
A resource Model is the abstract version of a model. Since a model just gathers and combines data, where does it get this data from? Thats where the resource models kick in. They represent the stage/adapter to different persistens storage mediums. In terms of abstraction, this is totally reasonable, because we can use our models and dont care about how the data is actually stored. 

The resource model takes care of this. 

So if our storage system changes, we just need to alter the resource model and use the rest of our code just as usual.
The last part of this topic are the collections. These are just array-like iterable collections (\*cough*) of models. Since models cannot be formed as an array, a collection does that. With `Mage::getModel('catalog/product')->getCollection()->load()` we would get an iterable, sortable, filterable set of all products in our shop as models.

### Configure a database connection

Though Magento keeps the initial database connection within its `local.xml` file in a `<db>` node, we can define our own database connection inside our `config.xml` file. The content of a `<db>` node looks like this:

	<db>
    	<table_prefix><![CDATA[]]></table_prefix>
   	</db>
       <default_setup>
           <connection>
                <host><![CDATA[127.0.0.1]]></host>
                <username><![CDATA[abc123]]></username>
                <password><![CDATA[abc123]]></password>
                <dbname><![CDATA[abc123]]></dbname>
                <active>1</active>
            </connection>
        </default_setup>
        
This is pretty self explaining, all neccessary information to establish a connection to a MySQL database is given in this snippet.

Besides the pure database access information, we can define read and write connectors, which magento can use (if we have more than one read and write to different databases) to split up read and write sessions to different databases. We could provide a second database to store all order information in - or build up a master/slave database system.

For this, Magento utilizes `<default_read>` and `<default_write>` as well as our own `<read>` and `<write>` nodes. In each of these nodes, we specify a `<connection>` with a defined database setup that should be used for this read/write access.

### Describe how Magento works with database tables

Help please! :)

### Describe the load-and-save process for a regular entity

With the `load()` and `save()` methods on models as well as on resource models, Magento tries to write the data into its storage. 
If we call one of the named methods on our model, Magento checks, wether the dataset of the model still exist and then executes the `_beforeSave()` method, which gives us the chance to perform some last minute changes on the models data.
When this is done, Magento tries to save the corresponding resource model	