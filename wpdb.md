## Wordpress Database (wpdb)

- [Use global default](#use-global-default)
- [Get Table Prefix](#get-table-prefix)
- [Get List Default Table without prefix](#get-list-default-table-without-prefix)
- [Usage](#usage)
  * [Get Var](#get-var)
  * [Get Row](#get-row)
  * [Get Col](#get-col)
  * [Get Result](#get-result)
  * [insert Row](#insert-row)
  * [Replace Row](#replace-row)
  * [Update Row](#update-row)
  * [Delete Row](#delete-row)
  * [Prepare and General Query](#prepare-and-general-query)
  * [Clear Cache query](#clear-cache-query)
  * [Check Last Error](#check-last-error)
- [Hook](#hook)
  * [Query Filter](#query-filter)
    + [example-> mysql insert ignore](#example---mysql-insert-ignore)
- [Custom Utility](#custom-utility)
  * [Save Query for Debug](#save-query-for-debug)
  * [Get List Column From mysql Table](#get-list-column-from-mysql-table)
  * [Get Column Comment in mysql table](#get-column-comment-in-mysql-table)
- [Connect To Server](#connect-to-server)
  * [Connect to Another Server](#connect-to-another-server)
  * [Connect another WP database and use function](#connect-another-wp-database-and-use-function)
- [Extra Package](#extra-package)
- [Reference](#reference)

<br />

### Use global default
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

### Get Table Prefix
```php
global $wpdb;
echo $wpdb->prefix;
echo $wpdb->prefix.'inbox';
````

### Get List Default Table without prefix
```php
global $wpdb;

$wpdb->posts
$wpdb->postmeta
$wpdb->comments
$wpdb->commentmeta
$wpdb->termmeta
$wpdb->terms
$wpdb->term_taxonomy
$wpdb->term_relationships
$wpdb->users
$wpdb->usermeta
$wpdb->links
$wpdb->options

```

### Usage

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

#### Get Col
```php
$postids = $wpdb->get_col( "SELECT `ID` from `wp_posts` ...");
foreach($postids as $id) {

}
```

#### Get Result
```php
$query = $wpdb->get_results( "SELECT ID, post_title FROM $wpdb->posts WHERE ...", ARRAY_A);
foreach ( $query as $row ) 
{
  setup_postdata( $row );
  echo get_the_title();
	echo $row['post_title];
}

//Count Number rows in last query
$wpdb->num_rows
//OR
count($query);
```

#### insert Row
```php
$wpdb->insert( 
	$wpdb->prefix.'table', 
	array( 
		'column1' => 'value1', 
		'column2' => 123 
	), 
	array( 
		'%s', 
		'%d' 
	) 
);

//Get Last ID
$wpdb->insert_id
```

> Possible format values: %s as string; %d as integer (whole number); and %f as float.

#### Replace Row
```php
//This function returns false if an existing row could not be replaced and a new row could not be inserted.

$wpdb->replace( 
	$wpdb->prefix.'table', 
	array( 
                'indexed_id' => 1,
		'column1' => 'value1', 
		'column2' => 123 
	), 
	array( 
                '%d',
		'%s', 
		'%d' 
	) 
);
```

#### Update Row
```php
$wpdb->update( 
	$wpdb->prefix.'table', 
	array( 
		'column1' => 'value1',	// string
		'column2' => 'value2'	// integer (number) 
	), 
	array( 'ID' => 1 ), 
	array( 
		'%s',	// value1
		'%d'	// value2
	), 
	array( '%d' ) 
);
```

#### Delete Row
```php
// Default usage.
$wpdb->delete( 'table', array( 'ID' => 1 ) );

// Using where formatting.
$wpdb->delete( 'table', array( 'ID' => 1 ), array( '%d' ) );
```

#### Prepare and General Query
```php
$wpdb->query( 
	$wpdb->prepare( "DELETE FROM $wpdb->postmeta WHERE post_id = %d AND meta_key = %s", 13, 'gargle' )
);
```

#### Clear Cache query
```php
$wpdb->flush();
//This clears $wpdb->last_result, $wpdb->last_query, and $wpdb->col_info.
```

#### Check Last Error
```php
//Output the error and add the error to the log
if($wpdb->last_error !== '') :
    $wpdb->print_error()
endif;

// Output the error but do not log it
function my_print_error(){
    global $wpdb;
    if($wpdb->last_error !== '') :
        $str   = htmlspecialchars( $wpdb->last_result, ENT_QUOTES );
        $query = htmlspecialchars( $wpdb->last_query, ENT_QUOTES );

        print "<div id='error'>
        <p class='wpdberror'><strong>WordPress database error:</strong> [$str]<br />
        <code>$query</code></p>
        </div>";
    endif;
}
```

### Hook

#### Query Filter
```php
add_filter('query', function_name');
function funxtion_name( $query ) {
	//Do Work
	return $query;
}
```

##### example-> mysql insert ignore
```php
//Mysql Ignore insert
function wp_ignore_insert( $query ) {
	$count = 0;
	$query = preg_replace( '/^(INSERT INTO)/i', 'INSERT IGNORE INTO', $query, 1, $count );
	return $query;
}

add_filter( 'query', 'wp_ignore_insert', 10 );
$wpdb->insert(
	$wpdb->prefix . 'table',
	array(
		'name' => '',
	),
	array( '%s' )
);
remove_filter( 'query', 'wp_ignore_insert', 10 );
```

### Custom Utility

#### Save Query for Debug
```php
//First, put this in wp-config.php:
define( 'SAVEQUERIES', true );

//Then in the footer of your theme put this:
global $wpdb;
echo "<pre>";
print_r( $wpdb->queries );
echo "</pre>";
```

#### Get List Column From mysql Table
```php
//First a mini query to database
global $wpdb;
$query = $wpdb->get_results( "SELECT * FROM `wp_posts` LIMIT 1");

//Get list of Column
$list = $wpdb->get_col_info('name');

//Flush at all
$wpdb->flush();

//Export the Top Code
Array
(
    [0] => ID
    [1] => post_author
    [2] => post_date
    [3] => post_date_gmt
    [4] => post_content
    [5] => post_title
    [6] => post_excerpt
    ...
)


//For Get Column name by Number Using
echo $wpdb->get_col_info('name', 2); //post_date

```

#### Get Column Comment in mysql table
```php
function wp_get_col_comment( $table_name, $column_name ) {
global $wpdb;
$field_comment = $wpdb->get_row("SELECT column_comment FROM information_schema.columns WHERE table_name = '`{$table_name}`' AND column_name LIKE '$column_name'", ARRAY_N );
return $field_comment[0];
}
```

### Connect To Server

#### Connect to Another Server
```php
$mydb = new wpdb('username','password','database','localhost');
$rows = $mydb->get_results("select Name from my_table");
```

#### Connect another WP database and use function
```php
function connect_to_site_DB(){
  global $wpdb;
  $wpdb = new wpdb( "username", "password", "database name", "localhost" );
  $wpdb->set_prefix('databasePrefix_');
  return $wpdb;
}

global $wpdb;
$wpdb_backup = $wpdb;
$wpdb = connect_to_site_DB();


//We set the WP_Query
$sitenews = new WP_Query(array(
  'post_type' => 'post',
  'post_status' => 'publish',
  'orderby' => 'menu_order',
  'order' => 'ASC',
  'no_found_rows' => true,
  'update_post_term_cache' => false, 
  'update_post_meta_cache' => false, 
  'ignore_sticky_posts' => true,
  'nopaging' => true,
  'posts_per_page' => 5,
));


// The Loop
if(!empty($sitenews->posts)) {
  $counter_posts = 1;
  foreach ($sitenews->posts as &$post){
    if ($counter_posts>5){
    break;
    }
    if($counter_posts==1) {
     include(locate_template('content-site-double.php', false, false));
    }
    else {
     include(locate_template('post-single.php', false, false));
    }
    $counter_posts++;
  }
}

//Here we close the connection
$wpdb = $wpdb_backup;
```

### Extra Package

| Package Name  |  Description |  Link  |
|---|---|---|
|  wp-activerecord |  Usign wpdb with Active Recoerd |  https://github.com/friedolinfoerder/wp-activerecord |
|  wp-eloquent | Eloquent ORM for WordPress  | https://github.com/tareq1988/wp-eloquent  |

### Reference
https://codex.wordpress.org/Class_Reference/wpdb
