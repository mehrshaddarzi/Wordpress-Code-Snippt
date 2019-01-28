#### use global default
```php
//Use in Function
global $wpdb;

//Use in Class
public $db;
public function __construct() {
  //Setup Wordpress Database
  $this->db = $GLOBALS['wpdb'];
  $this->db->get_var(....);
}
```

#### Connect to Another Server
```php
$mydb = new wpdb('username','password','database','localhost');
$rows = $mydb->get_results("select Name from my_table");
```

#### Get Var
```php
$user_count = $wpdb->get_var( "SELECT COUNT(*) FROM $wpdb->users" );
```

#### Get Row
```php
$mylink = $wpdb->get_row( "SELECT * FROM $wpdb->links WHERE link_id = 10", ARRAY_A );
if ( null !== $mylink ) {
  $name = $mylink['name'];
} else {
  // no link found
  return false;
}
```

> output_type One of three pre-defined constants. Defaults to OBJECT.<br />
OBJECT - result will be output as an object. e.g : $mylink->name; <br />
ARRAY_A - result will be output as an associative array. e.g : $mylink['name']; <br />
ARRAY_N - result will be output as a numerically indexed array.  e.g : $mylink[1];

