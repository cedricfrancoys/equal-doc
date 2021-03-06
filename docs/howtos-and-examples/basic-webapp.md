# A basic WebApp

## Learning by example how to create a web application with eQual

This tutorial details the required steps for creating a webapp from scratch using eQual.
As sample webapp, we are going to build a basic blog.

(Files from the "blog" demo package - SQL schema and dataset - as well as other sample files are available [[http://easyobject.cedricfrancoys.be/samples|here]].)

===== 0. Install easyObject =====

For installation notes, see [[http://easyobject.cedricfrancoys.be/wiki/install|easyObject quickstart]]

> URL ?

===== 1. Create a new package =====
**(Estimated time : 1 minute)**

This part is quite easy: in the //packages// folder, we create a new folder named "blog".
In addition, inside this new folder, let's create the following subfolders (as they are mandatory): "classes" and "views".

Tree structure is now:
```
  /
  /packages
    /blog
      /classes
      /views
```
===== 2. Write some classes =====
**(Estimated time : 3 minutes)**

Now, we need to create a new kind of object. Let's call it "post".

Each post consists of a title and some text (content). 

So,in the folder //packages/blog/classes//, we add a new file named //Post.class.php//.

```php
<?php

namespace blog {

    class Post extends \core\Object {
    
        public static function getColumns() {
            return array(
                'title'      => array('type' => 'string'),
                'content'    => array('type' => 'text')
            );
        }
    
    }
}
```


In addition, we want to be able to retrieve the name of the author of the post.
The special field //creator// gives us the id of the user (//core\User//) who created the post, but we would like to be able to display author's name without having to perform additional requests.


In order to do so, we add an //author// field, defined like this:
```php
'author'     => array(
                      'type'        => 'function', 
                      'result_type' => 'string', 
                      'store'       => true, 
                      'function'    => 'blog\Post::getAuthor'
                )
```

As well as the method allowing to retrieve the name of the given user id:
```php
public static function getAuthor($om, $uid, $oid, $lang) {
    $author = '';
    $res = $om->browse($uid, 'blog\Post', array($oid), array('creator'), $lang);
    if(is_array($res)) {
        $user_id = $res[$oid]['creator'];
        $res = $om->browse($uid, 'core\User', array($user_id), array('firstname', 'lastname'), $lang);
    }
    if(is_array($res)) $author = $res[$user_id]['firstname'].' '.$res[$user_id]['lastname'];
    return $author;
}

```


Finally, our file looks like this:

```php
<?php

namespace blog {

    class Post extends \core\Object {
    
        public static function getColumns() {
            return array(
                'title'      => array('type' => 'string'),
                'content'    => array('type' => 'text'),
                'author'     => array(
                                     'type' => 'function', 
                                     'result_type' => 'string', 
                                     'store' => true, 
                                     'function' => 'blog\Post::getAuthor'
                                )
            );
        }
    
        public static function getAuthor($om, $uid, $oid, $lang) {
            $author = '';
            $res = $om->browse($uid, 'blog\Post', array($oid), array('creator'), $lang);
            if(is_array($res)) {
                $user_id = $res[$oid]['creator'];
                $res = $om->browse($uid, 'core\User', array($user_id), array('firstname', 'lastname'), $lang);
            }
            if(is_array($res)) $author = $res[$user_id]['firstname'].' '.$res[$user_id]['lastname'];
            return $author;
        }
    
    }
}
```

Tree structure is now:
```
  /
  /packages
    /blog
      /classes
          Post.class.php      
      /views
```

===== 3. Create related DB tables =====
**(Estimated time : 2 minutes)**

Open your internet browser and go to the //core_utils// application. For instance, http://localhost/easyobject/index.php?show=core_utils.

(If you are getting confused, see easyObject [[http://easyobject.cedricfrancoys.be/wiki/url_mechanism|URL mechanism]].)

> URL ?

Now, among the packages list, select the newly created //blog// package, then choose the //sql-schema// plugin and click 'ok'.

In the right panel, you should see the following SQL code : 
```sql
CREATE TABLE IF NOT EXISTS `blog_post` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `created` datetime DEFAULT NULL,
    `modified` datetime DEFAULT NULL,
    `creator` int(11) NOT NULL DEFAULT '0',
    `modifier` int(11) NOT NULL DEFAULT '0',
    `published` tinyint(4) NOT NULL DEFAULT '0',
    `deleted` tinyint(4) NOT NULL DEFAULT '0',
    `title` varchar(255),
    `content` mediumtext,
    `author` mediumtext DEFAULT NULL,
    PRIMARY KEY (`id`)
) DEFAULT CHARSET=utf8;
```

You may now copy/paste this code in order to create a new table with your favorite SQL GUI manager (phpMyAdmin, workbench, ...)

===== 4. Create views =====
**(Estimated time : 1 minute)**

In the //packages/blog/views//, create two new files:
== 1. Post.form.default.html ==
```html
<form action="core_objects_update">
    <div>
        <span width="100%">
            <label for="title"></label><var id="title" required="true"></var>
        </span>	
        <div>
            <fieldset title="details">
                <div>
                    <section name="content">
                        <var id="content"></var>
                    </section>					
                </div>
            </fieldset>			
        </div>
    </div>
</form>
```
== 2. Post.list.default.html ==
```html
<ul>
    <li id="title" width="55%"></li>
    <li id="author" width="25%"></li>		
    <li id="created" width="20%"></li>			
</ul>
```

Tree structure is now:
```
  /
  /packages
    /blog
      /classes
        Post.class.php      
      /views
        Post.form.default.html
        Post.list.default.html
```

===== 5. Create some sample objects =====
**(Estimated time : 1 minute)**

  * Open your internet browser and go to the //core_manage// application. 
''For instanceb:
http://localhost/easyobject/index.php?show=core_manage''
  * Among packages list, select the //blog// package, then click on the //Post// class.
  * On the right panel, click on the 'create new' button.
  * Choose a title and a content for this new post
    For instance:
    Title : About this blog
    Content: Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Morbi vel erat non mauris convallis vehicula. Nulla et sapien. Integer tortor tellus, aliquam faucibus, convallis id, congue eu, quam. Mauris ullamcorper felis vitae erat. Proin feugiat, augue non elementum posuere, metus purus iaculis lectus, et tristique ligula justo vitae magna.



===== 6. Create an application =====
**(Estimated time : 2 minutes)**

==== 1. Template ====

  * To display some nice html, we need a template. Let's adapt one from a WP template designer.
  * Let's pick one from diovo.com : http://www.diovo.com/links/voidy/
  * We create a new folder //packages/blog/html/css// and copy //img// folder and //style.css// into it.
  * In //packages/blog/html// we put the adapted template below.
 Note that we added two //var// tags in order to display the template dynamically (based on the //post_id// parameter from the URL):
    <var id="content"></var>
and 
    <var id="recent_posts"></var>
    
    

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <meta charset="UTF-8">
    <title>my blog</title>
    <link media="all" rel="stylesheet" type="text/css" href="packages/blog/html/css/style.css" />
</head>
<body>
    <div id="body">
        <div id="header">
            <div id="logo">
                <div id="h1"><a href="#">Yet another easy blog</a></div>
                <div id="h2" style="font-style: italic;">powered by easyObject</div>        
            </div>
            <div id="header-icons"></div>
            <div id="menu">
                <div class="menu-bottom">
                    <ul><li class="page_item"><a href="#">contact</a></li></ul>
                    <div class="spacer" style="clear: both;"></div>
                </div>    
            </div>
        </div>
        <div id="main">    
            <div id="content">
                <div class="post type-post format-standard">
                    <var id="content"></var>            
                </div>                    
            </div>
            <div id="sidebar1" class="sidecol">
                <ul>
                    <li>
                        <p style="font-style: italic; font-family: Georgia,serif;">This blog is run by a hand-made webapp developed in minutes thanks to easyObject.</p>
                    </li>            
                    <li class="widget recent">
                        <h2 class="widgettitle">Latest articles</h2>
                        <ul><var id="recent_posts"></var></ul>                    
                    </li>
                </ul>
            </div>
            <div style="clear:both"></div>
        </div>    
        <div id="footer">
            <p>
                <a title="easyObject" href="http://easyobject.cedricfrancoys.be/">Powered by easyObject</a>
                &nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;
                <a title="Diovo" href="http://www.diovo.com/links/voidy/">Theme by Niyaz</a>
            </p>    
        </div>        
    </div>            
</body>
</html>
```

==== 2. Script ====

Finally, let's create a folder //packages/blog/apps// inside wich we put a file named //display.php// containing the code below.


```php
// the dispatcher (index.php) is in charge of setting the context and should include the easyObject library
defined('__EASYOBJECT_LIB') or die(__FILE__.' cannot be executed directly.');

// we'll need to format some dates
load_class('utils/DateFormatter');

// get the value of the post_id parameter (set it to 1 if not present), and put it in the $params array
$params = get_params(array('post_id'=>1));

/*
* A small html parser that replaces 'var' tags with their associated content.
*
* @param string $template    the full html code of a page, containing var tags to be replaced by content
* @param function $decorator    the function to use in order to return html code matching a var tag
*/
function decorate_template($template, $decorator) {
    $previous_pos = 0;
    $html = '';
    // use regular expression to locate all 'var' tags in the template
    preg_match_all("/<var([^>]*)>.*<\/var>/iU", $template, $matches, PREG_OFFSET_CAPTURE);
    // replace 'var' tags with their associated content
    for($i = 0, $j = count($matches[1]); $i < $j; ++$i) {
        // 1) get tag attributes
        $attributes = array();
        $args = explode(' ', ltrim($matches[1][$i][0]));
        foreach($args as $arg) {
            if(!strlen($arg)) continue;
            list($attribute, $value) = explode('=', $arg);
            $attributes[$attribute] = str_replace('"', '', $value);
        }
        // 2) get content pointed by var tag, replace tag with content and build resulting html
        $pos = $matches[0][$i][1];
        $len = strlen($matches[0][$i][0]);
        $html .= substr($template, $previous_pos, ($pos-$previous_pos)).$decorator($attributes);
        $previous_pos = $pos + $len;
    }
    // add trailer
    $html .= substr($template, $previous_pos);
    return $html;
}
/**
* Returns html part specified by $attributes (from a 'var' tag) and associated with current post id
* (here come the calls to easyObject API)
*
* @param array $attributes
*/
$get_html = function ($attributes) {
    global $params;
    $html = '';
    switch($attributes['id']) {
        case 'content':
            if(is_int($post_values = &browse('blog\Post', array($params['post_id']), array('id', 'created', 'title', 'content')))) break;
            $title = $post_values[$params['post_id']]['title'];
            $content = $post_values[$params['post_id']]['content'];
            $dateFormatter = new DateFormatter();
            $dateFormatter->setDate($post_values[$params['post_id']]['created'], DATE_TIME_SQL);
            $date = ucfirst(strftime("%A %d %B %Y", $dateFormatter->getTimestamp()));
            $html = "
                <h2 class=\"title\">$title</h2>
                <div class=\"meta\"><p>$date</p></div>
                <div class=\"entry\">$content</div>
            ";        
            break;
        case 'recent_posts':
            $ids = search('blog\Post', array(array(array())), 'created', 'desc', 0, 5);
            $recent_values = &browse('blog\Post', $ids, array('id', 'title'));
            foreach($recent_values as $values) {
                $title = $values['title'];
                $id = $values['id'];
                $html .= "<li><a href=\"index.php?show=blog_display&post_id={$id}\">$title</a></li>";
            }
            break;            
    }
    return $html;
};

// if we got the post_id and if the template file can be found, read the template and decorate it with current post values 
if(!is_null($params['post_id']) && file_exists('packages/blog/html/template.html')) print(decorate_template(file_get_contents('packages/blog/html/template.html'), $get_html));
```


Tree structure is now:
```
  /
  /packages
    /blog
      /apps
        display.php
      /classes
        Post.class.php      
      /html
        /css
          /img
          style.css
        template.html
      /views
        Post.form.default.html
        Post.list.default.html
```

To access your newly created blog, open your browser and request the //blog_display// application.
  URL example : http://localhost/easyobject/?show=blog_display

Remember that post_id will be set to 1 by default. To display another blog entry, you'll have to specify the related post_id in the URL.

  Another URL example could be : http://localhost/easyobject/?show=blog_display&post_id=2