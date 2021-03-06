# Directory structure

To understand the framework, here is what you need to know about its folders



## Root directory structure

```
/
├── bin
├── lib
├── packages
├── public
│   ├── .htaccess
│   └── index.php
├── spool
├── eq.lib.php
└── run.php
```



| File|  Description |
|-|-|
| .htaccess	        | Apache configuration file  used to prevent directory listing and handling url rewriting |
| eq.lib.php	| Boostrap library. Defines constants and global utilities |
| index.php	        | This script is also referred to as the dispatcher : its task is to include required libraries and to set the context. This is the main entry point.|
| run.php	| Server script for client-server mode|
| lib	        | Folder containing eQual library  (mostly classes and services definitions) |
| packages   | Folder containing installed packages (classes definition, translations, views, actions, …)|
| public   | Root public folder for the web server |
| bin   | Folder containing values of binary fields see BINARY_STORAGE_DIR |
| spool   | Dedicated to email sending |



The root folder is also the place where to place a `composer.json` file and the subsequent `/vendor` directory.

## public

| File        | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| .htaccess   | Apache configuration file  used to prevent directory listing and handling url rewriting |
| index.php   | This script is also referred to as the dispatcher : its task is to include required libraries and to set the context. This is the main entry point. |
| console.php | This is the only alternate entry point.                      |
| assets      |                                                              |
| assets/env  |                                                              |
| assets/i18n |                                                              |



The entry point of every project, you'll find **index.php** as well as the **packages** you will use



## config

​	**default.inc.php**, to configure your database and other parameters

​	**config-example.inc.php**, rename it as `config.inc.php` to use custom configuration

## lib

​	Libraries and services used as external resources

## run.php

​	Defines the DO, GET and SHOW behaviors for our queries, either by CLI, HTTP or PHP





## Packages

An application is divided in several parts, stored in a package folder located under the `/packages` directory.
Each package might contain the following folders (underlined folders are mandatory).

| Folder| Role | URI key   |  Examples           |
|-|-|-|-|
| **__classes__**    | model          |                  | `core\User.class.php`, `core\Group.class.php`, `core\Permission.class.php` |
| **__apps__**       | user interface (view) | show       | core_manage, core_utils |
| **actions**    | action handler (controller) | do       | core_manage, core_utils |
| **data**    | data provider | get       | core_objects_browse, core_user_lang |
| **__views__**    | templates |        |  |
| **i18n**    | translations |        |  |
| **assets** | static html |        | static content, javascripts, stylesheets, images |



​	Each **package** is defined as follows :

```
package_name
├── classes
│   └── */*.class.php
├── actions
│   └── */*.php
├── apps
│   └── */*.php
├── data
│   └── */*.php
├── tests
│   └── */*.php
├── init
│   └── *.json
├── views
│   ├── 
│   └── *.json
├── i18n
│   └── *
│       └── *.json
├── config.inc.php
└── readme.md
```

