### 1.0. Magento Architecture & Customisation Techniques
This section covers **33%** of the exam.

#### 1.1. Describe the Magento Module-Based Architecture
> What are the significant steps to add a new module?

You need to create the following files:
- `registration.php`
- `module.xml`

##### `registration.php`
This file is included by the Composer autoloader and is added to the static list of components in `Magento\Framework\Component\ComponentRegistrar`. Example:

```PHP
<?php
use \Magento\Framework\Component\ComponentRegistrar;

ComponentRegistrar::register(
    ComponentRegistrar::MODULE,        // type
    'YourCompany_YourModule',          // componentName
    __DIR__                            // path
);
```

Once Magento has found this file, it will then look to find `etc/module.xml`.

##### `module.xml`
A component declares itself (that is, defines its name and existence) in the `module.xml` file located in the module's `etc/` directory. This file specifies the loading sequence for the module and it's setup version (if not using Declarative Schema).

The sequence tells Magento which order to load modules in. This is done in `Magento\Framework\Module\ModuleList\Loader.php` and only occurs when the module is installed. This sequence is useful for events, plugins, preferences, and layouts.
- **NB:** If your module modifies the layout of another and your module is loaded first, the other module will overwrite your modifications.

Example:
```XML
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="YourCompany_YourModule" setup_version="1.0.0">
        <sequence>
            <module name="TheirCompany_TheirModule" />
            <module name="AnotherCompany_AnotherModule" />
        </sequence>
    </module>
</config>
```

---
> What are the different Composer package types?

| Name | Package Type | Description |
| :--: | :----------: | :---------- |
| Metapackage | metapackage | Technically, a Composer package type, not a Magento component type. A metapackage consists of only a `composer.json` file that specifies a list of components and their dependencies. For example, both Magento Open Source and Magento Commerce are metapackages. |
| Module | magento2-module | Code that modifies Magento application behavior. You can upload a single module to the Magento Marketplace or your module can be dependent on some parent package. |
| Theme | magento2-theme | Code that modifies the look and feel of the storefront or Magento Admin. |
| Language Package | magento2-language | Translations for the storefront or Admin. |

---
> When would you place a module in the app/code folder versus another location?

---
#### 1.2. Describe the Magento Directory Structure
> How do you locate different files in Magento?

All of the core Magento files are located in the `vendor/magento/` directory with some supporting JavaScript and CSS files being stored in `lib/`.

Third-party files can be found in their respective `vendor/vendor-name/module-name/` directories.

For your customisations:
- For modules, use `app/code`.
- For storefront themes, use `app/design/frontend`.
- For Admin themes, use `app/design/adminhtml`.
- For language packages, `use app/i18n`.

##### `/Api`: Service Contracts
The `/Api` directory stores the contracts or interfaces that are exposed to the API. An example of this would be `Magento\Catalog\Api\CategoryListInterface`.

##### `/Api/Data`: Data Service Contracts
This folder contains interfaces that represent data. Examples of this would be a
product interface, a category interface, or a customer interface. The concrete
implementations of these interfaces usually do little more than provide getters and
setters for data.

See `Magento\Catalog\Api\Data\ProductInterface` for an example.

##### `/Block`: View Models
Blocks can be considered as template assistants. Very little functionality or business logic should be done in templates - that is the responsibility of the Block.

##### `/Console`: Console Commands
When running `bin/magento` on the command line, a list of available commands
to run is output. The code for commands should be placed inside the `/Console` directory.

##### `/Controller`: Web Request Handlers
This directory stores controller classes for web requests.

##### `/Cron`: Cron Job Classes
Definitions for cron jobs are stored here.

##### `/etc`: Configuration Files
Any files directly within this directory are applied *globally*. You can restrict the *area* these files belong to by placing it in the relevant sub-directory - `etc/frontend/`, `etc/adminhtml/` etc.
- **NB:** Some files **MUST** be placed within a particular area whilst others **MUST** be global.

##### `/Helper`: Aggregated Functionality
Small, reusable snippets of code should be stored in a helper class.

##### `/i18n`: Localisation Files
This directory contains the translation CSV files for the module. These CSV files contain two columns, `from` and `to`.

##### `/Model`: Data Handling & Data Structures
A self-explanatory directory.

##### `/Model/ResourceModel`: Database Interactions
The classes stored in this directory dictate how data is stored and retrieved from the database. Any *direct* database interaction should be done within these files.

See `vendor/magento/module-catalog/Model/ResourceModel/Product.php` for an example.

##### `/Observer`: Event Listeners
When Magento fires an event, certain observers are called - decoupling the system. Magento Commerce integrates with RabbitMQ which allows even more control and reliability to this process. Event data should be able to be stored and then run in a queue at a later time.

Observers **MUST** implement the `Magento\Framework\Event\ObserverInterface`. The PHP class should follow the stand of using TitleCase while the event name should be snake_case.

Observers should not contain any business logic. This logic should instead be placed in another class and injected into your observer.

##### `/Plugin`
This directory stores your module's plugins. These are covered in more detail in a later section.

##### `/Setup`: Database Modification
Stores the following files:
- `InstallSchema.php`
    - Sets up table and column schema when the module is installed.
- `UpgradeSchema.php`
    - Modifies table and column schema when the module version is upgraded.
- `Recurring.php`
    - Runs after every install or upgrade.
- `InstallData.php`
    - Sets up data when the module is installed. An example would be adding a custom CMS block.
- `UpgradeData.php`
    - Modifies data after the module is installed and when the module version is upgraded.
- `RecurringData.php`
    - Applies to data after every install or upgrade.

##### `/Test`: Module Tests
This directory stores your module's tests. Tests can be run via the command line using `bin/magento dev:tests:run`.

##### `/Ui`: Data Generation Files

##### `/view/[area]/templates`: Block Templates
The counterpart to a Block is a template to render the HTML for the block. Whilst the block (in `/Block`) represents the business logic, the template represents how the results of the business logic are shown to the user.

##### `/view/[area]/web`: Web Assets
Web assets such as images, CSS, JavaScript, LESS, and SCSS are stored in this directory.

##### `view/[area]/web/template`: JavaScript Templates
HTML templates that can be requested asynchronously via JavaScript are stored in this directory. These files often contain KnockoutJS declarative bindings.

##### `/view/adminhtml/ui_component`: Adminhtml UI Components
This directory contains the XML configuration for UI components. They are used to represent distinct UI elements, such as grids and forms, and are designed to provide flexible user interface rendering. Most Magento Admin grids (such as the Catalog Product grid and the Customer grid) are composed with UI components. The
checkout on the frontend is also a UI component.

---
> What are the naming conventions, and how are namespaces established?

---
> How can you identify the files responsible for some functionality?

---
#### 1.3. Utilise Configuration and Configuration Variables Scope
> Which configuration files are important in the development cycle?

##### `acl.xml`
Defines permissions for accessing protected resources.

##### `config.xml`
Loads configuration values into `Stores > Configuration` in the Magento Admin. This file can also encrypt configuration entries.

##### `crontab.xml`
Defines cron job scheduling.

##### `di.xml`
This file configures dependency injection for your module. It defines plugins, preferences, concrete classes for interfaces, virtual types, and constructor argument modifications.

##### `email_templates.xml`

##### `events.xml`
This file registers observers.

##### `indexer.xml`
This file configures Magento indexers.

##### `module.xml`
This file is **required** by Magento.

##### `mview.xml`
Triggers a type of event when data is modified in a database column - most often used for indexing.

##### `view.xml`
This file is similar to `config.xml` but is used to specify the default values for design configuration.

##### `webapi.xml`
Configures API access and routes.

##### `widget.xml`
Configures widgets to be used in products, CMS blocks, and CMS pages.

##### `[area]/routes.xml`
This file tells Magento that this area accepts web requests. The route node configures the first part of the layout handle (route ID) and the front name (first segment in the URL after the domain name).

##### `adminhtml/menu.xml`
Configuration for the menu in Magento Admin.

##### `adminhtml/system.xml`
Configures tabs, sections, groups, and fields found in `Store > Configuration` in Magento Admin.

---
> How do you identify the configuration scope for a given variable?

---
> How do native Magento scopes (for example, price or inventory) affect development and decision-making processes?

---
> How can you fetch a system configuration value programmatically?

---
> How can you override system configuration values for a given store using XML configuration?

---
#### 1.4. Demonstrate How To Use Dependency Injection
> How are objects realized in code?
 
Since dependency injection happens automatically through the constructor, Magento
must handle class creation - either at the time of injection or via a factory.

##### Class Creation During Injection
First the object manager locates the proper class type. If an interface is requested, hopefully an entry in `di.xml` will provide a concrete class for the interface (if not, an exception will be thrown).

Then the parameters for the constructor are loaded and recursively parsed meaning that the dependencies for the initially requested class are loaded as well as the dependencies of those dependencies as well.

The deploy mode (`bin/magento deploy:mode:show)` determines which class loader is used:
- `vendor/magento/framework/ObjectManager/Factory/Dynamic/Developer.php`
- `vendor/magento/framework/ObjectManager/Factory/Dynamic/Production.php`

##### Class Creation Via Factories

---
> Why is it important to have a centralized object creation process?

Having a centralised process to create objects makes testing much easier. It also
provides a simple interface to substitute objects as well as modify existing ones.

---
> How can you override a native class, inject your class into another object, and use other techniques available in `di.xml` (for example, `virtualTypes`)?

##### Overriding Native Classes
Preferences are used to substitute entire classes. They can also be used to specify concrete classes for interfaces:
```XML
<preference for="Magento\GoogleTagManager\Block\ListJson"
            type="YourCompany\YourModule\Path\To\Your\Class"
/>
```

##### Injecting Your Class into Other Objects
Specify a `<type>` entry with your class as an `<argument>`:
```XML
<type name="Path\To\Your\Class\To\Inject\Into">
    <arguments>
        <argument xsi:type="object">
            Path\To\Your\Injected\Class
        </argument>
    </arguments>
</type>
```

##### Virtual Types
A virtual type allows you to create an instance of an existing class that has custom constructor arguments. This is useful in cases where you need a “new” class only because the constructor arguments need to be changed. This is used frequently in Magento to reduce redundant PHP classes.

---
> How would you obtain a class instance from different places in the code?

---
#### 1.5. Demonstrate Ability To Use Plugins
> How are plugins used in core code? How can they be used for customizations?

A plugin is a class that modifies the behavior of public class functions by intercepting a function call and running code before, after, or around that call. This allows you to customise or extend the behavior of original, public methods for any class or interface.

##### Before Plugin
Before plugins are used when you want to modify the function input. To modify the input of a function:
```PHP
public function beforeFunctionName(
    Class\Containing\The\Function $subject,
    $normalFunctionInput1,
    $normalFunctionInput2,
) {
    // ...
    
    return [$normalFunctionInput1, $normalFunctionInput2];
}
```

The `return` value determines the array of arguments being passed into the next plugin or targeted function.

##### After Plugin
After plugins are used when you want to modify the function output. To modify the output of a function:
```PHP
public function afterFunctionName(
    Class\Containing\The\Function $subject,
    $result
) {
    // ...
    
    return $result;
}
```

The `return` value determines the output of the function being passed into the next plugin or targeted function.

##### Around Plugin
Around plugins give you full control over the function, it's inputs, and it's output. To use an around plugin to replace a function:
```PHP
public function aroundFunctionName(
    Class\Containing\The\Function $subject,
    callable $proceed
) {
    // ...
    
    $result = $proceed();
    
    // ...
    
    return $result;
    
}
```

##### Declaring Plugins
To declare a plugin, add a `<type/>` to your module's `di.xml`:
```XML
<type name="Class\Containing\The\Function\To\Plug\Into">
    <plugin name="PluginName"
            type="Class\Containing\Your\Plugin"
            disabled="false"
            sortOrder="10"
    />
</type>
```

##### Plugin Limitations
Plugins have the following limitations:
- Only work with **public** functions
- **Do not** work with `final` classes or functions
- **Do not** work with `static` functions

---
#### 1.6. Configure Event Observers & Scheduled Jobs
Observers listen for events that are triggered in Magento. Scheduled jobs perform an action at a specified interval.

Observers **MUST** implement the `Magento\Framework\Event\ObserverInterface`.

---
> How are observers registered?

Create an `<event/>` node in your `etc/[area]/events.xml` file:
```XML
<event name="event_for_your_observer_to_listen_for">
    <observer name="observerName"
              instance="Your\Observer\Class"
    />
</event>
```

---
> How are they scoped for frontend or backend?

Place the `events.xml` file in the `etc/frontend/` and `etc/adminhtml/` directories respectively.

---
> How are automatic events created, and how should they be used?

Events should be used when you do not want to change the data. They can be triggered by injecting an instance of `Magento\Framework\Event\ManagerInterface` into the constructor and calling: `$this->eventManager->dispatch('event_name', [params]);`.

---
> How are scheduled jobs configured?

To configure a scheduled job you need to assign it a name, specify the function it should execute, the class that function belongs to, and set the schedule using the regular cron schedule notatation. Scheduled jobs are specified in the `etc/crontab.xml` file:
```XML
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Cron:etc/crontab.xsd">
    <group id="default">
        <job name="cron_job_name"
             instance="Path\To\Your\Class"
             method="execute">
            <schedule>*/5 * * * *</schedule>
        </job>
    </group>
</config>
```

---
#### 1.7. Utilise the CLI
> Which commands are available?

A full list of commands can be found by running `bin/magento`. All of the commands listed below can be run from within the `magento/` directory via the command line, prefixed with `bin/magento`.
```
Usage:
  command [options] [arguments]

Options:
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug

Available commands:
  help                                     Displays help for a command
  list                                     Lists commands
 admin
  admin:user:create                        Creates an administrator
  admin:user:unlock                        Unlock Admin Account
 app
  app:config:dump                          Create dump of application
  app:config:import                        Import data from shared configuration files to appropriate data storage
  app:config:status                        Checks if config propagation requires update
 cache
  cache:clean                              Cleans cache type(s)
  cache:disable                            Disables cache type(s)
  cache:enable                             Enables cache type(s)
  cache:flush                              Flushes cache storage used by cache type(s)
  cache:status                             Checks cache status
 catalog
  catalog:images:resize                    Creates resized product images
  catalog:product:attributes:cleanup       Removes unused product attributes.
 config
  config:sensitive:set                     Set sensitive configuration values
  config:set                               Change system configuration
  config:show                              Shows configuration value for given path. If path is not specified, all saved values will be shown
 cron
  cron:install                             Generates and installs crontab for current user
  cron:remove                              Removes tasks from crontab
  cron:run                                 Runs jobs by schedule
 customer
  customer:hash:upgrade                    Upgrade customer's hash according to the latest algorithm
 deploy
  deploy:mode:set                          Set application mode.
  deploy:mode:show                         Displays current application mode.
 dev
  dev:di:info                              Provides information on Dependency Injection configuration for the Command.
  dev:profiler:disable                     Disable the profiler.
  dev:profiler:enable                      Enable the profiler.
  dev:query-log:disable                    Disable DB query logging
  dev:query-log:enable                     Enable DB query logging
  dev:source-theme:deploy                  Collects and publishes source files for theme.
  dev:template-hints:disable               Disable frontend template hints. A cache flush might be required.
  dev:template-hints:enable                Enable frontend template hints. A cache flush might be required.
  dev:tests:run                            Runs tests
  dev:urn-catalog:generate                 Generates the catalog of URNs to *.xsd mappings for the IDE to highlight xml.
  dev:xml:convert                          Converts XML file using XSL style sheets
 encryption
  encryption:payment-data:update           Re-encrypts encrypted credit card data with latest encryption cipher.
 i18n
  i18n:collect-phrases                     Discovers phrases in the codebase
  i18n:pack                                Saves language package
  i18n:uninstall                           Uninstalls language packages
 indexer
  indexer:info                             Shows allowed Indexers
  indexer:reindex                          Reindexes Data
  indexer:reset                            Resets indexer status to invalid
  indexer:set-dimensions-mode              Set Indexer Dimensions Mode
  indexer:set-mode                         Sets index mode type
  indexer:show-dimensions-mode             Shows Indexer Dimension Mode
  indexer:show-mode                        Shows Index Mode
  indexer:status                           Shows status of Indexer
 info
  info:adminuri                            Displays the Magento Admin URI
  info:backups:list                        Prints list of available backup files
  info:currency:list                       Displays the list of available currencies
  info:dependencies:show-framework         Shows number of dependencies on Magento framework
  info:dependencies:show-modules           Shows number of dependencies between modules
  info:dependencies:show-modules-circular  Shows number of circular dependencies between modules
  info:language:list                       Displays the list of available language locales
  info:timezone:list                       Displays the list of available timezones
 maintenance
  maintenance:allow-ips                    Sets maintenance mode exempt IPs
  maintenance:disable                      Disables maintenance mode
  maintenance:enable                       Enables maintenance mode
  maintenance:status                       Displays maintenance mode status
 module
  module:disable                           Disables specified modules
  module:enable                            Enables specified modules
  module:status                            Displays status of modules
  module:uninstall                         Uninstalls modules installed by composer
 queue
  queue:consumers:list                     List of MessageQueue consumers
  queue:consumers:start                    Start MessageQueue consumer
 sampledata
  sampledata:deploy                        Deploy sample data modules for composer-based Magento installations
  sampledata:remove                        Remove all sample data packages from composer.json
  sampledata:reset                         Reset all sample data modules for re-installation
 setup
  setup:backup                             Takes backup of Magento Application code base, media and database
  setup:config:set                         Creates or modifies the deployment configuration
  setup:cron:run                           Runs cron job scheduled for setup application
  setup:db-data:upgrade                    Installs and upgrades data in the DB
  setup:db-declaration:generate-patch      Generate patch and put it in specific folder.
  setup:db-declaration:generate-whitelist  Generate whitelist of tables and columns that are allowed to be edited by declaration installer
  setup:db-schema:upgrade                  Installs and upgrades the DB schema
  setup:db:status                          Checks if DB schema or data requires upgrade
  setup:di:compile                         Generates DI configuration and all missing classes that can be auto-generated
  setup:install                            Installs the Magento application
  setup:performance:generate-fixtures      Generates fixtures
  setup:rollback                           Rolls back Magento Application codebase, media and database
  setup:static-content:deploy              Deploys static view files
  setup:store-config:set                   Installs the store configuration. Deprecated since 2.2.0. Use config:set instead
  setup:uninstall                          Uninstalls the Magento application
  setup:upgrade                            Upgrades the Magento application, DB data, and schema
 store
  store:list                               Displays the list of stores
  store:website:list                       Displays the list of websites
 theme
  theme:uninstall                          Uninstalls theme
```

---
> How are commands used in the development cycle?

Commands provide a secure method of perfoming tasks that may otherwise be insecure or more time consuming to perform through the Magento Admin. Some of the more commonly used commands during development are listd below:

##### `cache:disable`
Disables specific caches. You might use this to disable the `layout`, `block_html`, and `full_page` caches during the development of some frontend templates so Magento always generates the pages you are working on rather than older versions that it has cached. Example:
```
bin/magento cache:disable layout block_html full_page
```

##### `cache:flush`
Destroys the Magento caches, deleting the cache keys. This can be used to destroy any of the Magento caches in combination. Example:
```
# destroy all caches
bin/magento cache:flush

# destroy configuration cache
bin/magento cache:flush config
```

##### `cache:status`
Lists the caches and whether they are enabled or disabled. You might use this prior to using the `cache:disable` or `cache:enable` commands.

##### `deploy:mode:set`
Sets your deploy mode. During development you will likely want to set this to `developer` whilst on your live server you will want to set this to `production`. Example:
```
# developer mode
bin/magento deploy:mode:set developer

# production mode
bin/magento deploy:mode:set production

# default mode
bin/magento deploy:mode:set default
```

##### `dev:query-log:enable`
Enables database query logging.

##### `indexer:reindex`
Runs the Magento indexers.

##### `module:disable`
Disables a specified module. Example:
```
bin/magento module:disable Module_Name
```

##### `module:enable`
Enables a specified module. Example:
```
bin/magento module:enable Module_Name
```

##### `module:status`
List all modules and whether they are enabled or disabled.

##### `setup:db:status`
Checks whether the Magento database needs to be upgraded.

##### `setup:upgrade`
Synchronises module versions in the database with those defined in the codebase.
