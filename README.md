InfluxDB [![Build Status](https://travis-ci.org/crodas/InfluxPHP.png?branch=master)](https://travis-ci.org/crodas/InfluxPHP)
========

Simple PHP client for [InfluxDB](http://influxdb.org/), an open-source, distributed, time series, events, and metrics database with no external dependencies.

How to install it
-----------------

The easiest way is to install it via [composer](http://getcomposer.org)

```bash
composer require crodas/influx-php:\*
```

How to use it
-------------

You need to create a client object.

```php
$client = new \crodas\InfluxPHP\Client(
   "localhost" /*default*/,
   8086 /* default */,
   "root" /* by default */,
   "root" /* by default */
);
```

The first time you should create an database.

```php
$db = $client->createDatabase("foobar");
$client->createUser("foo", "bar"); // <-- create user/password
```

You can create more users than the default root user. 

```php
$client->createUser('username', "password");
```

Show existing users:


```php
$users = $client->getUsers();
```

User privileges are controlled by per-databases users. Any user can have 'read', 'write' or 'all' access, represented by the constants InfluxClient::PRIV_READ, InfluxClient::PRIV_WRITE and InfluxClient::PRIV_ALL.


```php
$client->grantPrivilege(InfluxClient::PRIV_ALL, 'database', 'username');

// revoke rights by
$client->revokePrivilege(InfluxClient::PRIV_ALL, 'database', 'username');
```

The cluster administrator can be set with:

```php
$client->setAdmin('username');

// delete admin righty by:
$client->deleteAdmin('einuser');
```

Create data is very simple.

```php
$db = $client->foobar;

// single input:
$db->insert("some label", [ 'fields' => array('value' => 2)]); 

// single input; the 'name' identifier will be overwritten by array content:
$db->insert("some label", ['name'=> 'some label', 'fields' => array('value' => 2)]);

// multiple insert, this is better...
// The 'name' field is optional, if not set, the default parameter (here: 'foobar') will be used

$db->insert("foobar", array(
            array('name' => 'lala',   'fields' => array('type' => 'foobar', 'karma' => 25)),
            array(   'fields' => array('type' => 'foobar', 'karma' => 45)),
            ));

```

It is recommended that you encode most metadata into the series Tags. 

```php

$db->insert("foobar",[['tags' => ['type' => 'one'], 'fields' => ['value' => 10]]]);

```


Now you can get the database object and start querying.

```php
$db = $client->foobar;
// OR
$db = $client->getDatabase("foobar");

foreach ($db->query("SELECT * FROM foo") as $row) {
    var_dump($row, $row->time);
}
```

If there are multiple result series, a MultipleResultSeriesObject instance will be returned. So if you are not sure about the results of the query, check the type of the returned object. 

An example of getting multiple result sets is:

```php

// Here the 'type' value is stored as metadata in the tags entry. So if there are two 'type' tags found, you will get two result series
$result = $db->query("SELECT count(value) FROM test1 where  time >= '2015-01-01T12:00:00Z' and time < '2015-01-02T00:00:00Z' group by time(1h), type");

```


Please have a look at the DBTest.php class, you will find some more examples there. 
