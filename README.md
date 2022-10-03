# PHP Query Builder

An easy to use sql query builder written in pure php >=5.6

installation:
  require_once 'DB.php';
  $hostname = "localhost";
  $database = "test";
  $username = "root";
  $password = "";
  $db = new DB($hostname, $username, $password, $database);
  
usage:
  $res = $db->table("products")->select()->get();
