# PHP Query Builder

An easy to use sql query builder written in pure php >=5.6

### Setup:
```
<?php
require_once 'DB.php';
$hostname = "localhost";
$database = "test";
$username = "root";
$password = "";
$db = new DB($hostname, $username, $password, $database);
```
### Usage:
```
$res = $db->table("products")->select()->get(); // returns an array of product table records with all coulmns included
```
