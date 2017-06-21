ArangoDb Extension for Yii 2
===========================

This extension provides the [ArangoDB](http://www.arangodb.org/) integration for the Yii2 framework.


Installation
------------

This extension requires [ArangoDB PHP Extension](https://github.com/triAGENS/ArangoDB-PHP)

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```
php composer.phar require --prefer-dist explosivebit/yii2-arangodb "*"
```

or add

```
"explosivebit/yii2-arangodb": "*"
```

to the require section of your composer.json.


General Usage
-------------

To use this extension, simply add the following code in your application configuration:

```php
return [
    //....
    'components' => [
        'arangodb' => [
            'class' => '\explosivebit\arangodb\Connection',
            'connectionOptions' => [
                triagens\ArangoDb\ConnectionOptions::OPTION_DATABASE => "mydatabase",
                triagens\ArangoDb\ConnectionOptions::OPTION_ENDPOINT => 'tcp://127.0.0.1:8529',
                triagens\ArangoDb\ConnectionOptions::OPTION_AUTH_TYPE => 'Basic',
                //triagens\ArangoDb\ConnectionOptions::OPTION_AUTH_USER   => '',
                //triagens\ArangoDb\ConnectionOptions::OPTION_AUTH_PASSWD => '',
            ],
        ],
    ],
];
```

Using the connection instance you may access databases, collections and documents.

To perform "find" queries, you should use [[\explosivebit\arangodb\Query]]:

```php
use explosivebit\arangodb\Query;

$query = new Query;
// compose the query
$query->select(['name', 'status'])
    ->from('customer')
    ->limit(10);
// execute the query
$rows = $query->all();
```


Using the ArangoDB ActiveRecord
------------------------------

This extension provides ActiveRecord solution similar ot the [[\yii\db\ActiveRecord]].
To declare an ActiveRecord class you need to extend [[\explosivebit\arangodb\ActiveRecord]] and
implement the `collectionName` and 'attributes' methods:

```php
use explosivebit\arangodb\ActiveRecord;

class Customer extends ActiveRecord
{
    /**
     * @return string the name of the index associated with this ActiveRecord class.
     */
    public static function collectionName()
    {
        return 'customer';
    }

    /**
     * @return array list of attribute names.
     */
    public function attributes()
    {
        return ['_key', 'name', 'email', 'address', 'status'];
    }
}
```

Note: collection primary key name ('_key') should be always explicitly setup as an attribute.

You can use [[\yii\data\ActiveDataProvider]] with [[\explosivebit\arangodb\Query]] and [[\explosivebit\arangodb\ActiveQuery]]:

```php
use yii\data\ActiveDataProvider;
use explosivebit\arangodb\Query;

$query = new Query;
$query->from('customer')->where(['status' => 2]);
$provider = new ActiveDataProvider([
    'query' => $query,
    'pagination' => [
        'pageSize' => 10,
    ]
]);
$models = $provider->getModels();
```

```php
use yii\data\ActiveDataProvider;
use app\models\Customer;

$provider = new ActiveDataProvider([
    'query' => Customer::find(),
    'pagination' => [
        'pageSize' => 10,
    ]
]);
$models = $provider->getModels();
```


Using Migrations
----------------

ArangoDB migrations are managed via [[explosivebit\arangodb\console\controllers\MigrateController]], which is an analog of regular
[[\yii\console\controllers\MigrateController]].

In order to enable this command you should adjust the configuration of your console application:

```php
return [
    // ...
    'controllerMap' => [
        'arangodb-migrate' => 'explosivebit\arangodb\console\controllers\MigrateController'
    ],
];
```

Below are some common usages of this command:

```
# creates a new migration named 'create_user_collection'
yii arangodb-migrate/create create_user_collection

# applies ALL new migrations
yii arangodb-migrate

# Apply 1 migration
yii arangodb-migrate/up 1

# reverts the last applied migration
yii arangodb-migrate/down

# Rollback 1 migration
yii arangodb-migrate/down 1
```

Once migration created, you can configure the migration:

When you start the migration as in the example below, you create a normal document collection with the migration name, you need to add an additional parameter `Type => 3` to create the `edge collection`. Example of such a query: `$this-> createCollection ('services', ['Type' => 3] );` If you want to create a document collection, then delete the `'Type' => 3` or replace the number 3 with 2.


```php
class m170413_210957_create_services_collection extends \explosivebit\arangodb\Migration
{
    public function up()
    {
        # When you start the migration, a collection of "services" with the edge type is created
        $this->createCollection('services',['Type' => 3]);
    }

    public function down()
    {
        # When the migration rollback starts, the collection "services"
        $this->dropCollection('services');
    }
}
```


Using Debug Panel
-----------------

Add ArangoDb panel to your yii\debug\Module configuration

```php
return [
    'bootstrap' => ['debug'],
    'modules' => [
        'debug' => 'yii\debug\Module',
        'panels' => [
            'arango' => [
                'class' => 'explosivebit\arangodb\panels\arangodb\ArangoDbPanel',
            ],
        ],
        ...
    ],
    ...
];
```


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/DevGroup-ru/yii2-arangodb/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

