# Views

Views are intended to describe how to present the objects to end-users under a given context.
They are used as templates for the front-end, and are stored as JSON files within the `views` folder of their respective package.

Each of them represents a mode of visualization: form, list, chart, dashboard, etc; and can be edited independently from the models they relate to. 

It is possible to define as many views (of different types, or variations of same type) as necessary. 
Each view is referenced by an ID, which is composed of its type and its name.

As a convention, a default view for `list` and `form` types should be defined for each entity.

**The generic filename format** is: `{class_name}.{view_type}.{view_name}.json`

* `class_name`: the class name of the entity the view relates to (e.g. default form view for  `core\User` is stored as `packages/core/views/User.form.default.json`)
* `view_type`: Possible values are :'*list*', '*form*','*chart*','*dashboard*'
* `view_name`: As a convention, classes should always have a 'default' view for types 'list' and 'view'.



Here is a recap for the `core\User` entity :

| FILENAME           | ENTITY | VIEW TYPE | VIEW NAME | VIEW ID |
| --------------------------------- | --------- | --------- | --------- | --------- |
| `core\views\User.list.default.json` | core\User | list | default | list.default |
| `core\views\User.form.default.json` | core\User | form | default | form.default |





## Front-end logic

A **view** relates to an entity and has a type and a name. The view itself requests the corresponding data from the server (template or translation) when loading the layout at which a domain can be specified.
Within a view, a layout defines the way in which the widgets are linked to the model. The view is synchronized with the model during modifications. 

Keep in mind that if the view's class extends another class, which will be called the parent, then it should also contain all the fields from this parent class except the computed ones and the ones that are already present in this child class.

A **Model** is a collection of objects of a given entity. This class keeps the full schema of the entity with the default values that are then updated by this model after it requests the corresponding data from the server.

A **Layout** is the layout associated with a given view. It is always linked to a Model.

A **Widget** is responsible for displaying the value of an object's field (in 'view' or 'edit' mode). It synchronizes its value with the Model to which it is associated via the Layout and the View that is using it.

!!! note "About widget property"
    The <em>widget</em> property has various field options:     
    * ```view```: to indicate which view template to use with the widget.  
    * ```header```: (boolean) to emphasise the widget. When set to true, the field is considered as header and is shown with a bigger font-size.  
    * ```readonly```: (boolean). When set to true, the field is displayed as read-only (can't be changed).  
    For one2many and many2many field, it is also possible:     
    * to specify the order and limit the loaded lines shown in auto-complete  
    * to force using a specific widget



## Views commons
Some attributes are common to all types of views. Below is a list of the common attributes and their role.

### Structure summary

| PROPERTY        | DESCRIPTION                                                  |
| --------------- | ------------------------------------------------------------ |
| **name**        | The **name** property is mandatory and relates to the unique name assigned to the view. |
| **description** | A **description** property allows to give a short hint about the view's context or the way it is intended to be used. |
| **access**      | (optional)                                                   |
| **actions**     | (optional)                                                   |
| **controller**      | (optional) When set, the **controller** property allows to customize the controller that is used for populating the view (by default: 'model_collect' for lists, 'model_read' for forms). |
| **header**      | (optional) In the header property, one can customize the standard buttons of the header and the actions attached to these. |

### name

### description

### access

groups: array (list of groups the view is restricted to)

Example : 

```
    "access": ['root']
```

### actions <a id="view_commons_actions"></a>
The optional **actions**  property  contains a list of objects defining a custom list of possible actions attached to the view.

Each action item  relates to a button, displayed in the right side of the header, which, when clicked, relays a request to a given controller. Upon successful completion of the action, the view is automatically refreshed.

| PROPERTY | DESCRIPTION                                           |
| --------------- | ------------------------------------------------------------ |
| **id**          | Identifier of the action for translation purpose (can be set in the i18n related file). |
| **description** | The description that is displayed to the user when (s)he clicks on the related button. |
| **label**       | Label assigned to the view.                                  |
| **controller**  | Controller to invoke when the user confirms the action. By default, the `id` of the current object is sent as a parameter. |
| **visible**     | (optional) Domain (array) of conditions to meet in order to make the action button visible. Example: `"visible": ["status", "=", "quote"]` |
| **confirm**     | (optional) If set to true, a confirmation dialog is displayed before relaying the request to the controller. |
| **params**      | (optional) Associative array mapping fields with their values. Values can be assigned by referencing a property of the current user (e.g. `user.login`) or current object (for form views). |

Example:

```json
"actions": [
  {
    "id": "action.option",
    "label": "Set as Option",
    "description": "Rental units will be blocked but no funding will be claimed yet.",
    "controller": "lodging_booking_option",
    "visible": ["status", "=", "quote"],
    "confirm": true,
    "params": {
        "id": "object.id"
    }
  },
  {
    "id": "action.validate",
    "label": "Confirm Booking",
    "description": "Rental units will be blocked and the invoicing plan will be set up.",
    "controller": "lodging_booking_confirm",
    "visible": ["status", "in", ["quote", "option"]]
  },
  {
    "id": "action.checkin",
    "label": "Check In",
    "description": "The host has arrived: the rental units will be marked as occupied.",
    "controller": "lodging_booking_checkin",
    "visible": ["status", "=", "validated"]
  },
  {
    "id": "action.checkout",
    "label": "Check Out",
    "description": "The host is leaving: the rental units will be marked for cleaning.",
    "controller": "lodging_booking_checkout",
    "visible": ["status", "=", "checkedin"]
  },
  {
    "id": "action.invoice",
    "label": "Invoice",
    "description": "The host has left and room has been reviewed: emit the invoice.",
    "controller": "lodging_booking_invoice",
    "visible": ["status", "=", "checkedout"]
  }
]
```



If target controller requires one or more parameter, the view will generate a dialog asking the user for the values to be assigned to each parameter.

If no parameter is required but the `confirm` property is set to `true`, then a confirmation dialog is displayed before performing the action.

Here below is a flow diagram that recaps the interactions between the controller and the `confirm` property.

<center>
<img src="https://files.yesbabylon.org/document/2196871ef46f435e9e899287fe4c1256" />
</center>


### controller <a id="view_commons_controller"></a>

The optional **controller**  property specifies the controller to use for requesting the Model to use for populating the View (either a single object or a collection of objects).

The default values is (core_)model_collect

!!! Note
    Controller are considered as entities. When a controller is specified for a list View, a related `search.default` view is expected , which describes the layout of the form for values relating to fields returned by the `::announce` method of the view controller. In turn, those values are sent to the controller along with default values (`entity`, `fields`, `domain`, `order`, `sort`, `start`, `limit`, `lang`) for feeding the View. Example: for controller `sale_booking_collect`, a `packages/sale/views/booking/collect.search.default.json` file is expected.



### header <a id="view_commons_header"></a>

The **header** section allows to override the default behavior of the view.

#### Structure summary

| PROPERTY    | DESCRIPTION                                                  |
| ----------- | ------------------------------------------------------------ |
| **actions** | This property allows to customize the actions buttons shown in the left part of the View header. |
| **visible** | (optional) boolean or array (domain)                         |


##### actions

The actions property can be used : to force action buttons visibility; to define the order of the actions for buttons having multiple actions ("split buttons"); and to override the configuration of the subsequent Views (for relational fields).

* For **forms**, default actions are : `ACTION.EDIT`, `ACTION.SAVE`, `ACTION.CREATE`, `ACTION.CANCEL`
* For **lists**, default actions are : `ACTION.SELECT`, `ACTION.CREATE`

Each action item is either a boolean or an array of items describing the order of the buttons and parameters for subsequent views. 

Empty arrays or items set to false mean that the action is not available for the View. If action is an array with multiple items, the related button will be displayed as a split-button (note: in that case, the order of the items is maintained).

| ACTION | DESCRIPTION | ID(S) |
| ---- | ---- |---- |
| **ACTION.EDIT** | For forms in view mode, allows to edit the current object. ||
| **ACTION.SAVE** | For forms in edit mode, **ACTION.SAVE** is the action used for storing the changes made to the current object. |`SAVE_AND_CLOSE`, `SAVE_AND_VIEW`, `SAVE_AND_CONTINUE`|
| **ACTION.CREATE** | For all views, **ACTION.CREATE** is the action used for creating a new current object. |`CREATE`, `ADD`|
| **ACTION.CANCEL** | For forms in edit mode, allows to cancel the changes made to the current object. |`CANCEL_AND_CLOSE`, `CANCEL_AND_VIEW`|
| **ACTION.SELECT** | For relational fields, allows to select or add one or many objects and relay selection to parent View. ||

**Predefined actions**: 


| ACTION | DESCRIPTION | CONTROLLER |
| ---- | ---- | ---- |
| SAVE_AND_CLOSE | The view is saved and close. The user is brought to the previous context. | |
| SAVE_AND_CONTINUE | The view is saved and left open allowing the user to perform further changes. A snackbar is given as feedback to the user. | |
| SAVE_AND_VIEW | The view is saved and closed. The user is brought to the 'view' version (SAVE action can only occur in 'edit' mode). In most cases, the 'view' version is the parent. If not, a new (similar) context is opened in 'view' mode. | |
| SAVE_AND_EDIT | The view is saved and closed. The user is brought to a cloned context (still in 'edit' mode). | |
| CREATE |  | model_create |
| ADD |  | model_update |

**Usage example**: 

```json
    "header": {
        "actions": {
            "ACTION.CREATE": [
                {
                    "view": "form.create",
                    "description": "Overload form to use for objects creation.",
                    "domain": ["parent_status", "=", "object.status"]
                }
            ],
            "ACTION.SELECT": false,
            "ACTION.SAVE": [ {"id": "SAVE_AND_CONTINUE"}, {"id": "SAVE_AND_CLOSE"} ],
            "ACTION.CANCEL": [ {"id": CANCEL_AND_VIEW"}]
        }
    }
```



##### visible





## Form views

**Forms** allow to view and edit individual objects. It is possible to define as many views as desired, and a given entity should always have default form view (`{entity}.form.default.json`). 

Forms views are JSON objects that describe how to render a specific view related to a given entity.

### Minimal example

Example.form.default.json
```json
{
  "name": "Example",
  "description": "Simple form for displaying a basic objects",
  "actions": [],
  "layout": {
    "groups": [    
      {
        "label": "",
        "sections": [
          {
            "rows": [
              {
                "columns": [
                  {
                    "width": "50%",
                    "items": [
                      {
                          "type": "field",
                          "value": "id",
                          "width": "50%"
                      },
                      {
                          "type": "field",
                          "value": "name",
                          "width": "50%"
                      }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```



### Structure summary

| PROPERTY | DESCRIPTION                                 |
| ----------- | ------------------------------------------------------------ |
| **name**    | The **name** property is mandatory and relates to the unique name assigned to the view. |
| **description** | A **description** property allows to give a short hint about the way the view is intended to be used. |
|**header**|This section allows to override action buttons that are displayed in the header of the form.|
|**actions**|(optional) A list of actions associated to a view. If set, visible actions (see below) will be shown in the right-part of the header.|



### header

The **header** property is common to all views. For details about its structure see <a href="#view_commons_header">views commons</a>.



### actions

The **action** property is common to all views. For details about its structure see <a href="#view_commons_actions">views commons</a>.



### layout

The layout part holds a nested structure that describes the way the (form) view has to be rendered (which fields, using which widgets) and how to place its elements, by grouping fields within rows and columns.



#### layout.groups

The groups are stacked vertically. A layout must always have at least 1 group.

|PROPERTY|DESCRIPTION|
|--|--|
|**label**|name of the group|
|**sections**|Array of sections objects. A group must always have at least 1 section.|



#### group.sections

A group must always have at least 1 section.

When several sections are present, each section is displayed under a tabs.

|PROPERTY|DESCRIPTION|
|--|--|
|**label**|(optional) Label (en) of the section. The label of a section is only displayed when there are several sections.|
|**id**|(optional) identifier for mapping the section in translation files|
|**visible**|(optional) a domain conditioning the visibility of the section and its tab (ex. `["status", "not in", ["quote", "option"]]`)|
|**rows**|Array of rows objects. A section must always have at least 1 row.|



#### section.rows

|PROPERTY|DESCRIPTION|
|--|--|
|**columns**|An array of columns objects that should be displayed within the row.|



#### row.columns

|PROPERTY|DESCRIPTION|
|--|--|
|**width**|Width of the column, as percentage of the width of the parent row (ex.: "25%").|
|**items**|An array of items that should be displayer within the column.|



#### column.items

Each column has a list of items, which are element describing which fields are to be rendered, how to render them (room within the column, widget override, ...), and under what conditions they must be displayed.

Each item is an object accepting the following properties : 

|Property|Description|
|--|--|
|**label**|(optional) Default label|
|**type**||
|**value**||
|**width**|width, in percentage of the parent column width.|
|**visible**|(optional) either a boolean (true, false) or a domain (ex. `["is_complete", "=", true]` )|
|**domain**|(optional) (ex. `["type", "<>", "I"]`)|
|**widget**|(optional) additional settings to apply on the widget that holds the fields|



#### item.widget

Within item`objects`, the widget property allows to refine the configuration of the widget (i.e. how the widget has to be rendered within the view).

|PROPERTY|DESCRIPTION|
|--|--|
|**heading**|(optional) if set to true, the widget is emphasized|
|**readonly**|(optional) if set to true, the value cannot be modified (marked as disabled in edit mode). If the readonly property is set to true in the schema, it cannot be overriden.|

Some additional properties apply only to specific field types. Here is the full list of the available options by type of field:

|FIELD TYPE|PROPERTY||
|-|-|-|
|`many2many`, `one2many`|||
|| **header**       |(optional) The widget can override the configuration that will be relayed to the subsequent View for M2M and O2M fields. For details about **header** structure see <a href="#view_commons_header">views commons</a>.|
||**header.actions**|(optional) works the same way as View **actions** property. For details about **header** structure see <a href="#view_commons_header">views commons</a>.|
||**view**|(optional) ID of the view to use for subobjects. For forms, default is "form.default", and for lists, default is "list.default" (ex.: "form.create").|
||**domain**||
|`many2one`|||
||**order**|Name of the field which the collection must be sorted on.|
||**sort**|Direction for sorting 'asc' (ascending order) or 'desc' (descending order).|
||**limit**||
||**domain**||

**Example:**

```
"widget": {
    "header": {
        "actions": {
            "ACTION.SELECT": true,
            "ACTION.CREATE": false        
        }
    }
}
```





!!! Note
    When an `usage` property is set in the schema of the entity, the widget is adapted accordingly. For example, when a field has its **type** set as `float` and its **usage** set to `amount/percent`, in view mode, it is displayed as an integer value between 0 an 100, and followed by a '%' sign. For instance "0.12" is converted to "'12%'').



### Real life example

A real example of a form view is shown below, which is the Category form of a package having multiple `sections` (tabs) each having a label(Categories, Product Models and Booking Types) and an id(`section.categories_id`, `section.product_models`, `section.booking_types`) to be able to be translated in using the "i18n". The field called `name` has a `widget` property with an attribute `header` set to true which makes it have a bigger font as mentioned earlier and the most important field of this view.

The view's name is `Category.form.default.json` and is as follows:

```json
{
  "name": "Category",
  "description": "Categories are not related to Families and allow a different way of grouping Products.",
  "layout": {
    "groups": [
      {
        "sections": [
          {
            "label": "Categories",
            "id": "section.categories_id",
            "rows": [
              {
                "columns": [
                  {
                    "width": "50%",
                    "items": [
                      {
                        "type": "field",
                        "value": "name",
                        "width": "100%",
                        "widget": {
                            "header": true
                        }
                      },
                      {
                        "type": "field",
                        "value": "description",
                        "width": "100%"
                      }
                    ]
                  }
                ]
              }
            ]
          },
          {
            "label": "Product Models",
            "id": "section.product_models",
            "rows": [
              {
                "columns": [
                  {
                    "width": "100%",
                    "items": [
                      {
                        "type": "field",
                        "value": "product_models_ids",
                        "width": "100%"
                      }
                    ]
                  }
                ]
              }
            ]
          },
          {
            "label": "Booking Types",
            "id": "section.booking_types",
            "rows": [
              {
                "columns": [
                  {
                    "width": "100%",
                    "items": [
                      {
                          "type": "field",
                          "value": "booking_types_ids",
                          "width": "100%"
                      }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```



## List views

List views are used to display collections of items. It contains the same properties mentioned in the ```Form View``` section, such as `name`, `description`, `layout` and a few additional ones. Some of the additional properties that are specific for the list views are `filters`, `pager` for displaying in navigation bar, `selection_actions` which allows the modification of an object in the list or exporting and printing it. ```sortable``` property could also be added to the list view, which enables the action of sorting the list when it has the value "true" since it's of type Boolean.
By clicking on one row in the list, it redirects you to the editable form related to the view. 



### Minimal example

The name of file is displayed like so: `packages/core/views/User.list.default.json`
A list view is defined according to the following structure:

```json
{
  "name": "",
  "description": "",
  "domain": [],
  "filters": [
    {
        "id": "lang.french",
        "label": "fran??ais",
        "description": "Users with locale set to french",
        "clause": ["language", "=", "fr"] 
    }
  ],
  "layout": {
    "filter": "bool",
    "pager": "bool",
    "selection_actions": [
      {
          "label": "export",
          "action": "products_export"
      }
    ],
    "items": [
        {
            "type": "field",
            "value": "id",
            "width": "10%",
            "sortable": true,
            "readonly": true
        },
        {
            "type": "field",
            "value": "created",
            "width": "25%",
            "sortable": true
        },
        {
            "type": "field",
            "value": "validated",
            "width": "10%"
        },
        {
            "type": "field",
            "value": "login",
            "widget": {
                "link": true
            },
            "width": "30%",
            "sortable": true
        },
        {
            "type": "field",
            "value": "language",
            "width": "10%",
            "widget": {
                "type": "select",
                "values": ["fr", "en", "nl"]
            }
        },
        {
            "type": "field",
            "value": "groups_ids",
            "label": "Groups",
            "width": "0%",
            "visible": false,
            "widget": {
                "type": "one2many"
            }
        }    
    ]
  }
}
```

The list view of the Category form mentioned in the above section contains the `name` of the list which is Categories, a `description` and the main fields to be displayed for quick use like the "name" and the "description" of the categories.
The list view is named *Category.list.default.json* and has the following structure:

```json
{
    "name": "Categories",
     "description": "This view is intended for displaying the list of categories.",
     "layout": {
         "items": [
            {
                 "type": "field",
                 "value": "name",
                 "width": "15%"
            },
            {
                 "type": "field",
                 "value": "description",
                 "width": "25%"
            }
        ]
    }
}
```



### Structure summary

### name



### description

### group_by

A `group_by` array can be set to describe the way the objects have to be grouped.
Each item in the array is either a field name or the descriptor of an operation to perform on a specific field.

Example : 

```json
    "group_by": ["date"]
```



The operations items have the following structure : 

```json
{ 
    "field": "product_id", 
    "operation": ["SUM", "object.qty"]
}
```

Another example : 

```json
    "group_by": ["date", {"field": "product_id", "operation": ["SUM", "object.qty"]}]
```



### order

String holding the name(s) of the field to sort results on, separated with commas.
Example : 

```json
    "order": "sku,product_model_id"
```


### sort

String litteral ('*desc*' or '*asc*')

Example:
```json
    "sort": "asc"
```



### limit

integer (max size of result set)

Example : 

```json
    "limit": 100
```

Bear in mind that the default controller (core_mode_collect), has a `max` constraint of `500` for this parameter.

### domain



### filters

The **filter** property allows to provide a series of predefined search filters   

```json
"filters": [
    {
        "id": "lang.french",
        "label": "French",
        "description": "French speaking people",
        "clause": ["language", "=", "fr"] 
    }
]
```

### header
The **header** property is common to all views. For details about its structure see <a href="#view_commons_header">views commons</a>.



### actions

The **action** property is common to all views. For details about its structure see <a href="#view_commons_actions">views commons</a>.



### exports

Printing a document such as a contract can be done in the `list` view. Multiple fields will have to be added such as the `id` of the contract, the `label`, the `icon` of the printer also known as "print" is added. Also, a small `description`, a `controller` having the value "model_export-print" used to trigger the printing action, the `view` which corresponds to the specific view "print.default" and finally `visible` field should be displayed as well. 

All these fields are added inside of the <em>exports</em> section of list view, which is an array of objects that has the below structure:

```json
"exports": [
        {
            "id": "export.print.contract",
            "label": "Print contract",
            "icon": "print",
            "description": "Print contract related to the booking.",
            "controller": "lodging_booking_print-contract",
            "view": "print.default",
            "visible": ["status", "=", "quote"]
        }
    ]
```

The view property points to an HTML file that will be parsed and filled with selected object values before being converted to PDF.  



### layout

The layout part holds an structure that describes the way the (list) view has to be rendered (which fields, using which widgets) and how to order its elements, group them or apply operations on them.



#### layout.items

The list view consists of a table having a series of columns (items). Each column relates to a field, and is described by an item that specifies how the field is to be rendered, the behaviors attached to it (ordering, sorting, ...), and under what conditions it must be displayed.

Each item is an object accepting the following properties : 

| PROPERTY | DESCRIPTION                                           |
| --------------- | ------------------------------------------------------------ |
| label           | (optional) Default label                                     |
| type            | always 'field'                                               |
| value           | the name of the field                                        |
| width           | column width, in percentage of the list width.               |
| visible         | (optional) either a boolean (true, false) or a domain (ex. `["is_complete", "=", true]` ) |
| sortable        | (optional) boolean to mark the column related to the field as sortable. |



### operations

This property allows to apply a series of operations on one or more columns, for the displayed records set.

Each entry of the `opearations` object associates a name (ID of an operation - which will allow to group the results), with an associative array mapping field names with operation descriptors.

In turn, each descriptor accepts the following properties : 

| PROPERTY  | DESCRIPTION                                                  |
| --------- | ------------------------------------------------------------ |
| operation | The operation to apply on the related field (see operation syntax below). |
| usage     | The `usage` of the operation as hint for displaying the result. (see) Example : `amount/money:2`, `numeric/integer` |
| suffix    | (optional) string to append to the result.                   |
| prefix    | (optional) string to prepend to the result.                  |



Examples: 

```json
"operations": {
    "total": {
        "total_paid": {
            "id": "operations.total.total_paid",
            "label": "Total received",
            "operation": "SUM",
            "usage": "amount/money:2"
        },
        "total_due": {
            "operation": "SUM",
            "usage": "amount/money:2"
        }
    }
}
```



```json
"operations": {
    "total": {
        "rental_unit_id": {
           "operation": "COUNT",
           "usage": "numeric/integer",
           "suffix": "p."
        }
    }
}
```



##### Operation Syntax

```
Unary operators : [ OPERATOR, {FIELD | OPERATION} ]
```
OR
```
Binary operators : [ OPERATOR, {FIELD | OPERATION}, {FIELD | OPERATION} ]
```

##### Binary operators
|OPERATOR|RESULT|SYNTAX|
|--|--|--|
|+|Sum of `a` and `b`.|`['+', a, b]`|
|-|Difference between `a` and `b`.|`['-', a, b]`|
|*|Product of `a` by `b`.|`['*', a, b]`|
|/|Division of `a` by `b`.|`['/', a, b]`|
|%|Modulo `b` of `a`.|`['%', a, b]`|
|^|`a` at power  `b`.|`['^', a, b]`|

##### Unary operators

|OPERATOR|SYNTAX|
|--|--|
|SUM|`['SUM', object.field]`|
|AVG|`['AVG', object.field]` (which is a shortcut for `['/', ['SUM', object.field], ['COUNT', object.field]]`)|
|COUNT|`['COUNT', object.field]`|
|MIN|`['MIN', object.field]`|
|MAX|`['MAX', object.field]`|



## Print views



## Menu Views

Menus allow to define custom tree structures of action-buttons for accessing specific routes or contexts.

Menu items have the following structure : 

| Property        | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| **id**          | Identifier of the item (used for translations).              |
| **label**       | Title of the item to display within the menu.                |
| **description** | (optional) Short string explaining the purpose of the item (the view it leads to). |
| **icon**        | (optional) icon to show aside the item.                      |
| **type**        | (mandatory) either 'entry' or 'parent'. In case an item is a 'parent', it also have a 'children' property. |



Parent items have a **children** property, which is an array holding a list of items (which, in turn, can be either parents or entries).

Entries items have a **context** property, which has the following structure : 




| Property   | Description                                              |
| ---------- | -------------------------------------------------------- |
| **entity** | Entity to which relates the view to show.                |
| **view**   | ID of the view to use for showing the targeted entities. |
| **order**  | (optional)                                               |
| **sort**   | (optional)                                               |
| **domain** | (optional) Domain to apply to specified view.            |

Example:

```json
"id": "item.pos_sessions",
"label": "Sessions",
"description": "",
"icon": "menu_book",
"type": "parent",
"children": [
    {
        "id": "item.pos_sessions.pending",
        "type": "entry",
        "label": "Pending sessions",
        "description": "", 
        "context": {
            "entity": "lodging\\sale\\pos\\CashdeskSession",
            "view": "list.default",
            "order": "created",
            "sort": "desc",
            "domain": [ ["status", "=", "pending"], ["center_id", "in", "user.centers_ids"] ]
        }
    }
]
```



As other views, a menu has a `name` property and a `layout` property, that describes how the items are going to be displayed. 

In the example shown below, one parent menu item is present named "New Booking" and it contains two children, "New Booking" to create a new booking and "All Bookings" that displays the list of all the bookings ordered by id and sorted in descending order.

```json
{
    "name": "Booking menu",
    "layout": {
        "items": [
            {
                "id": "item.bookings",
                "label": "Bookings",
                "description": "",
                "icon": "menu_book",
                "type": "parent",
                "children": [
                    {
                        "id": "item.new_booking",
                        "type": "entry",
                        "label": "New booking",
                        "description": "",
                        "icon": "add",
                        "context": {
                            "entity": "lodging\\sale\\booking\\Booking",
                            "view": "form.default",
                            "purpose": "create"
                        }
                    },
                    {
                        "id": "item.all_booking",
                        "type": "entry",
                        "label": "All bookings",
                        "description": "",
                        "context": {
                            "entity": "lodging\\sale\\booking\\Booking",
                            "view": "list.default",
                            "order": "id",
                            "sort": "desc"
                        }
                    }
                ]
            }
        ]
    }
}
```

## Dashboard Views

```json
{
    "name": "Main dashboard",
    "layout": {
        "groups": [
            {
                "label": "",
                "height": "100%",
                "sections": [
                    {                
                        "rows": [
                            {
                                "height": "50%",
                                "columns": [
                                    {
                                        "width": "50%",
                                        "items": [
                                            {
                                                "id": "item.bookings",
                                                "label": "Bookings",
                                                "description": "",
                                                "width": "50%",
                                                "entity": "",
                                                "view": ""
                                            }
                                        ]
                                    }
                                ]
                            }
                        ]
                    }
                ]
            }            
        ]
    }
}
```
