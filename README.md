# PHP Query Builder

An easy to use sql query builder written in pure php >=5.6

<a name="index_block"></a>

* [1. Setup](#block1)
* [2. Basic Usage](#block2.1)
    * [2.1. SELECT](#block2.1)
    * [2.2. INSERT](#block2.2)
    * [2.3. UPDATE](#block2.3)
    * [2.4. DELETE](#block2.4)
    * [2.5. RAW](#block2.5)
* [3. Advanced Usage](#block3.1)
    * [3.1 WHERE](#block3.1)
    * [3.2 JOIN](#block3.2)
    * [3.3 Transaction](#block3.3)
* [4. Error Handling](#block4.1)
    * [4.1 setDebug](#block4.1)
    * [4.2 setException](#block4.2)
    * [4.3 setLog](#block4.3)
    * [4.4 getError](#block4.4)
    * [4.5 getSql](#block4.5)
* [5. TODO](#block5.1)

<a name="block1"></a>
### Setup: [↑](#index_block)
---
Initialize `DB` class with required parameters and you're ready to go.
```php
<?php
require_once 'DB.php';
$hostname = "localhost";
$database = "test_db";
$username = "root";
$password = "";
$db = new ORM($hostname, $username, $password, $database);
```
### Usage:
```php
$res = $db->table("products")->select()->execute(); // returns an array of `products` table records with all coulmns included
var_dump($res);
```
#### Output:
```php
array (size=2)
  0 => 
    array (size=3)
      'id' => string '1' (length=1)
      'name' => string 'test product 01' (length=15)
      'price' => string '25$' (length=3)
  1 => 
    array (size=3)
      'id' => string '2' (length=1)
      'name' => string 'test product 02' (length=15)
      'price' => string '36$' (length=3)
```
<a name="block2.1"></a>
### SELECT: [↑](#index_block)
---
You can choose which columns should be returned.
```php
$db->table("products")->select("id", "price")->execute();
```
Leave select empty to get all columns.
```php
$db->table("products")->select()->execute();
```
<a name="block2.2"></a>
### INSERT: [↑](#index_block)
---
```php
$res = $db->table('products')->insert([
    "name"=>"product_test",
    "price"=>25,
])->execute();
var_dump($res);
```
You can refer to the returned result to ensure that the insert was successful.

Also you can find inserted row ID in the returned result if you need.
#### Output:
```php
array (size=2)
  'status' => int 1 // on success
  'result' => int 2 // inserted row ID
```
<a name="block2.3"></a>
### UPDATE: [↑](#index_block)
---
```php
$res = $db->table('products')->update([
    'price'=>40
])->where([
    ["id", "=", 2]
])->execute();
var_dump($res);
```
You can refer to the returned result to ensure that the update was successful.
#### Output:
```php
array (size=1)
  'status' => int 1 // on success
```
<a name="block2.4"></a>
### DELETE: [↑](#index_block)
---
```php
$res = $db->table("products")->delete()->where([
    ["id", "=", 2]
])->execute();
var_dump($res);
```
You can refer to the returned result to ensure that the delete was successful.
#### Output:
```php
array (size=1)
  'status' => int 1 // on success
```
<a name="block2.5"></a>
### RAW: [↑](#index_block)
---
You can always use RAW method to execute your own SQL code for complex cases
```php
$res = $db->raw("SELECT * FROM `products`")->execute();
```
The return result will be quite similar to the previous methods, depending on your raw operation method
<a name="block3.1"></a>
### WHERE: [↑](#index_block)
---
You can filter records with WHERE method.

#### Single where:
```php
$db->table("products")->select()->where([
    ["id", "=", 3],
    ["name", "!=", "null"]
])->execute();
```
#### SQL equalivent:
> SELECT * FROM products WHERE ((id = 3) AND (name != 'null'));
#### Multiple where:
```php
$db->table("products")->select()->where([
    ["id", "=", 3],
    ["name", "!=", "null"]
])->orWhere([
    ["id", "<", 3],
    ["name", "=", "null"]
])->execute();
```
#### SQL equalivent:
> SELECT * FROM products WHERE ((id = 3) AND (name != 'null')) OR ((id < 3) OR (name = 'null'));
#### A bit more advanced way of filtering:
```php
$IDs = [10, 20];
$db->table("products")->select()
    ->whereIn([
        "id", [1, 2, 3]
    ])->orWhere(function ($query) {
        $query->whereNotIn([
            "id", [4, 5]
        ])->andWhere([
            ["name", "like", "S%"], // name starts with S
            ["id", "=", 13]
        ]);
    })->andWhere(function ($query) use ($IDs) {
        $query->whereBetween("id", [10, 20]);
    })->execute();
```
#### SQL equalivent:
> SELECT * FROM products WHERE (id IN (1,2,3)) OR ((id NOT IN (4,5)) AND ((name like 'S%') AND (id = 13))) AND ((id BETWEEN 10 AND 20));
<a name="block3.2"></a>
### JOIN: [↑](#index_block)
---
#### Inner join:
```php
$db->table("products")
    ->select()
    ->join("cart", "products.id", "=", "cart.products_id")
    ->execute();
```
#### SQL equalivent:
> SELECT * FROM products INNER JOIN cart ON products.id=cart.products_id;
#### Left join:
```php
$db->table("products")
    ->select()
    ->leftJoin("cart", "products.id", "=", "cart.products_id")
    ->execute();
```
#### SQL equalivent:
> SELECT * FROM products LEFT JOIN cart ON products.id=cart.products_id;
#### Right join:
```php
$db->table("products")
    ->select()
    ->rightJoin("cart", "products.id", "=", "cart.products_id")
    ->execute();
```
#### SQL equalivent:
> SELECT * FROM products RIGHT JOIN cart ON products.id=cart.products_id;
<a name="block3.3"></a>
### Transaction: [↑](#index_block)
---
Handle your sensitive sequence of operations with Transaction method
#### Example:
```php
$db->transaction(); // START transaction
try{
    $product = $db->table('products')->insert([
        "name"=>"product_test59",
        "price"=>25,
    ])->execute();
    $db->table('cart')->insert([
        "products_id"=>$product["result"],
        "count"=>10,
    ])->execute();
    $db->commit(); // END transaction
}catch (Exception $ex) {
    $db->rollback(); // END transaction
    var_dump($ex->getMessage());
}
```
<a name="block4.1"></a>
### setDebug: [↑](#index_block)
---
By default if there is any kind of error. The program ignores it and tries to avoid the crash. Although you can change how it works with the help of the `setDebug`. When `setDebug` is turned on, the program will shut down as soon as it encounters an error.
#### Configuration:
```php
$db->setDebug(true);
// do your desired operations below here
```
<a name="block4.2"></a>
### setException: [↑](#index_block)
---
If you need to throw an exception without shutting down the program as soon as an error occurs, turn `setException` on.

Just don't forget to use try catch otherwise you will get fatal error.
#### Configuration:
```php
$db->setException(true);
try {
    // do your desired operations here
}catch (Exception $ex) {
    var_dump($ex->getMessage());
}
```
<a name="block4.3"></a>
### setLog: [↑](#index_block)
---
If you need to store errors as a log file; turn `setLog` on.

Default location of log file is under your SERVER_ROOT directory in a folder called `log`.
#### Configuration:
```php
// use this method right after the DB class initilized
$db->setLog(true, $_SERVER['DOCUMENT_ROOT'] . '/newPath/');
```
<a name="block4.4"></a>
### getError: [↑](#index_block)
---
Use `getError` to list current errors.
#### Usage:
```php
$db->table("products")->select("id")->execute();
var_dump($db->getError()); // returns false if there was no error
```
<a name="block4.5"></a>
### getSql: [↑](#index_block)
---
Use `getSql` to list current SQL raw statements.
#### Usage:
```php
$db->table("products")->select("id")->execute();
var_dump($db->getSql());
```
#### Output:
```php
array (size=1)
  0 => string '`products`# SELECT `id` FROM `products`;' (length=40)
```
<a name="block5.1"></a>
### TODO: [↑](#index_block)
---
* Support other famous databases like PostgreSQL, MongoDB
* Support PDO
* Optimize Where method
