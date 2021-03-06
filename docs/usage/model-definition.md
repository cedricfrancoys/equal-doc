# Model definition



Each Model is defined in a `.class.php` file , located in the `/packages/{package_name}/classes` file of the package it relates to (see [Directory Structure](directory-structure.md)), and inherits from a common ancestor: the `Model` class, declared in the `equal\orm` namespace and defined in file `/lib/equal/orm/Model.class.php`.

A class is always referred to as an **entity** which holds the package it belongs to (packages are used as namespaces).

The syntax is : `package_name\class_name` (ex. :`core\setting\SettingValue`).



A class consists of a series of fields definition, along with specific methods.



Each `field` definition may contain one or more of these properties: 

* 
* `alias`: which represents another name that the field in known for. For example: field name is "name" so the alias would be "display_name".
* 
* `result_type`: Mandatory when type is '*computed*'. It specifies the type of the result of this specific field, which can be any of the mentioned typed in the `type` property.
* 
* ```default```: is the default value of the field.
* `description` (optional): is a small brief about the field.
* `onchange` (optional): calls a function to get it's value whenever a change exists.
* ```ondelete```: has 2 values, either <em>cascade</em> or <em>null</em>. 
    * <em>cascade</em> is used in the context of on delete action for the parent class, delete this field too. 
    * <em>null</em> is used to identify that when deleting the parent class, the current field shouldn't be delete with it.
* ```ondetach```: has one value which is <em>delete</em>, which is triggered when the update event for a field is happening. 
* `selection`: represents all the options in a field of a class. It's like the list of possible options in a dropdown menu.
* `visible`: It specifies the conditions that must be met in order for the field to be relevant.
* `foreign_object`: is the path to the parent class of a field in another one.
* `foreign_field`: the name of the field that refers to the parent class.
* `required`: Marks a field as mandatory (storing an object without giving a value for that field will raise an error).
* ```domain```: is a condition set for a field which implies for example that a field is equal, or in the range of, or different than another one. It is written like so: ```'domain' => ['relationship', '=', 'customer']```. This means that the specific field that has this domain must have a relationship type equal to customer.
* ```usage```: it specifies under which format the field is used. For example: 
    * markup/html
    * country/iso-3166:2 (for the country address)
    * amount/money (for the price)
    * phone, email, url
    * uri/urn:iban (for the account iban)
    * amount/percent (for the rate)
    * date/year:4 (for the year)
    * uri/urn:ean (for the ean code)
    * language/iso-639:2 (for the language abbreviation like fr, en, du...)


Below is an example of a class called Category having multiple fields for which we will then show how to write its ```Form View``` and ```List View```.

```php
<?php 
class Category extends Model {
    
    public static function getColumns() {
      return [
        'name' => [
          'type'              => 'string',
          'description'       => "Name of the category (for all variants).",
          'required'          => true
        ],

        'description' => [
          'type'              => 'string',
          'description'       => "A few details about category purpose and usage."
        ],
        
        'product_models_ids' => [ 
          'type'              => 'many2many', 
          'foreign_object'    => 'sale\catalog\ProductModel', 
          'foreign_field'     => 'categories_ids', 
          'rel_table'         => 'sale_product_rel_productmodel_category', 
          'rel_foreign_key'   => 'productmodel_id',
          'rel_local_key'     => 'category_id',
          'description'       => 'List of product models assigned to the category.'
        ]

      ];
    }
}
```



##  Classes

Objects are grouped into classes that define the structure and methods of those objects. 

Classes definitions from a same package are placed into the folder //packages/[package_name]/classes//.

Here is an example of the class definition syntax :
```php
<?php
namespace school;
use equal\orm\Model;

class Student extends Model {
  public static function getColumns() {
    return [
      'firstname'  => ['type' => 'string'],
      'lastname'   => ['type' => 'string'],
      'birthdate'  => ['type' => 'date']
    ];
  }
}
```



## Consistency with Database

Consistency between models (`*.class.php`files) and database schema must be maintained at all time, and types must be compatibles (ex.: a varchar(255) column in the database may represent a string, as well as a short_text or a text in the related class).

For each entity, a table is defined in the database that has a structure matching the related class definition. 

Each time a new class is created or the schema of a class is modified, the SQL schema must be adapted consequently.
Action controller `core_init_package` and Data provider `utils_sql-schema` are made to help with this task.

Also, Action controller `core_test_package-consistency` can help to spot any incompatibilily or inconsistancy in the definition the classes from a given package.


## Fields types

### Basic fields

These fields values are directly stored in the associated SQL table and don't need to be processed.

Basic fields are: **boolean**, **integer**, **float**, **string**, **text**, **array**, **date**, **time**, **datetime**, **file**, **binary**, **many2one**



#### boolean

Numeric value of Boolean type (**true** or **false**)

!!! Note
	For booleans, we use the PHP built-in constant : true and false (when using '0', '1', 0 or 1, value is converted to a bool).

#### integer

Signed numeric value (negative or positive)

> With PHP it depends on the platform (generally 32 bits signed), with SQL it depends on the chosen type and size

#### float

Floating point numeric value (float or double)

#### string

Short string without formatting/carriage returns (ex: lastname, place, ???)

#### text

Text potentially long and including formatting

> notes : with SQL, the types TEXT, BLOB or MEDIUMTEXT, MEDIUMBLOB are recommended 

#### binary

Any binary value (ex : a picture, a document, ???)

#### date, time & datetime

When stored to the DBMS, those types follow the standard SQL format:

* date: YYYY-mm-dd
* time: HH:mm:ss
* datetime: YYYY-mm-dd HH:mm:ss

!!! Tip 
	Internally, dates, times and datetimes are handled as timestamps. These are adapted to SQL format or JSON format when needed.

#### many2one
Relational field used for fields holding a N-1 relation, that is to say a numeric value that represents the identifier of the pointed object.

N-1 relation, generating an integer to identify the related one2many object


	foreign_object -> select amongst other classes
		+ ability to create symetrical one (one2many)
```php
// todolist\Class
// Each task has only 1 user assigned at a time
'user_id' => [
    'type'           => 'many2one',
    'foreign_object' => 'todolist\User'
]
```



### Complex fields

Those fields require additional processing to be retrieved

Complex fields include , **one2many**, **many2many**, **computed**



#### one2many

1-N relation, related to a many2one object

		foreign_object -> select amongst other classes
		foreign_field -> select amongst fields of other class
				+ ability to create a new one (many2many) : temporary creation in one of both ways !
		rel_table -> check amongst existing rel tables ({package}_rel_local_foreign or rel_foreign_local)
				if no table yet -> create
		rel_local_key -> {local_class}_id
		rel_foreign_key -> {foreign_class}_id


```php
// todolist\User
// Each user can have many tasks
'tasks_ids'  => [
    'type'           => 'one2many',
    'foreign_object' => 'todolist\Task',
    'foreign_field'  => 'user_id'
]
```

#### many2many

M-N relation

	foreign_object -> select amongst other classes
		+ ability to create symetrical one (one2many)
```php
// school\Teacher
'courses_ids' => [
    'type' 			  => 'many2many', 
    'foreign_object'  => 'school\Course', 
    'foreign_field'	  => 'teachers_ids', 
    'rel_table'		  => 'school_rel_course_teacher', 
    'rel_foreign_key' => 'course_id', 
    'rel_local_key'	  => 'teacher_id'
]
```

#### computed

	type: 'computed'
	'function': any
	result_type : select ('boolean', 'integer', 'float', 'string', 'text', 'html' )
	store : boolean
The 'computed' type doesn't exist directly inside the DB. 

To get it,  we use the 'function' key, that will point at any function.

 It will return a processed value and afterwards, it can be stored inside the DB.

> Most of the time the use of the store attribute requires that field(s) on which depends the computed value, has an onchange event, triggering the update of the calculated field (see example). 

When trying to load a computed field, 

  * if 'store' is not defined or set to false, it computes the value using the provided method each time the field value is requested
  * if 'store' is set to true and the field isn't in the DB (NULL), it computes the value using the provided method
  * if 'store' is set to true and the field is in the DB, it uses the DB value

```php
// Example from core\Permission
// ...
    'rights' => [
        'type'		   => 'integer',
        'onchange'	   => 'core\Permission::onchangeRights'
    ],
    'rights_txt' => [
        'type'		   => 'computed', 
        'store'		   => true, 
        'result_type'  => 'string', 
        'function'	   => 'core\Permission::getRightsTxt'
    ],
// ...


public static function onchangeRights($om, $ids, $lang) {
    $rights = Permission::getRightsTxt($om, $ids, $lang);
    foreach($ids as $oid) { 
        $om->write('core\Permission', $oid, ['rights_txt' => $rights[$oid]], $lang);
    }
}

public static function getRightsTxt($om, $ids, $lang) {
    $res = array();
    $values = $om->read('core\Permission', $ids, array('rights'), $lang);
    foreach($ids as $oid) {
        $rights_txt = array();
        $rights = $values[$oid]['rights'];
        if($rights & QN_R_CREATE)   $rights_txt[] = 'create';
        if($rights & QN_R_READ)     $rights_txt[] = 'read';
        if($rights & QN_R_WRITE)    $rights_txt[] = 'write';
        if($rights & QN_R_DELETE)   $rights_txt[] = 'delete';
        if($rights & QN_R_MANAGE)   $rights_txt[] = 'manage';
        $res[$oid] = implode(', ', $rights_txt);
    }
    return $res;
}
```





## Fields attributes

| Type         | Attribute   |Usage                    |
| - | -|-|
| ***Basic fields*** |  | |
|  | type        | Type of the field. Accepted types are: *alias*, *computed*, *many2one*, *many2many*, *one2many*, *integer*, *string*, *float*, *boolean*, *text*, *date* and *datetime*. |
|              | multilang | (optional) boolean telling if current field can be translated (default value: false) [More info](i18n.md) |
|              | onchange | (optional) string holding the name of the method to invoke when field is updated, with format : `package\Class::method` (method is in charge of updating fields that depends on current field)<br/>The signature of this method is: ```public static function onchangeName($orm, $oids, $lang) {}``` |
|              | selection | (optional). Value selected from a pre-defined list   <a href="#anchor">Example</a> |
| **many2one** |  |  |
|     | foreign_object     | class toward which current field is pointing back |
| **one2many** |  |  |
|     | foreign_object     | class toward which current field is pointing back |
|     | foreign_field     | name of the field of the pointed class that is pointing back toward the current class        |
|     | foreign_key     | (optional) field that serves as identifier for objects pointed by the relation (default value: 'id') |
|     | order     | (optional) field on which pointed objects must be sorted |
|     | sort     | (optional) direction fort sorting (possible values: 'asc', 'desc') |
| **many2many** |  |  |
|     | foreign_object     | class toward which is pointing the current field        |
|     | foreign_field | name of the field of the pointed class that is pointing back toward the current class |
|     | rel_table | name of the SQL table dedicated to the m2m relation (recommended syntax: `package_rel_class1_class2`) |
|     | rel_local_key | name of the column in `rel_table` holding the identifier of the current object |
|     | rel_foreign_key | name of the column in de `rel_table` holding the identifier of the pointed object |
| **computed** |  |  |
|  | result_type     | type of the value resulting from the invoked function |
|     | store     | (optional) boolean telling if the result must be stored in database (in that case, the related table must contain a column for the field). By default, this attribute is set to **false**. |
|     | function  | String holding the name of the method to invoke for computing the value of the field. Syntax: `method` (relative to current class) or `package\Class::method` (absolute notation) |
|              | multilang | (optional) boolean telling if field can be translated (default value: false). This applies only for stored fields. |



<a name="anchor">Example of a basic field using the `selection` attribute:</a> 

```php 
'fieldname' => [
    'type'      => 'string',
    'selection' => ['choice1', 'choice2','choice3']
]
```

### onupdate method





### ondelete method



### System fields

Some fields are mandatory, and defined in the `Model` class.

| Name     | type           | Usage                                                        |
| -------- | -------------- | ------------------------------------------------------------ |
| id       | integer        | Unique identifier of the object.                             |
| name     | string         | Name to use when referring to the object (in views). By default, this field is an alias of  `id`. |
| status   | string         | 'draft', 'instance', 'archive'                               |
| created  | datetime       |                                                              |
| creator  | foreign_object | `core\User`                                                  |
| modified | datetime       |                                                              |
| modifier | foreign_object | `core\User`                                                  |

### Custom methods