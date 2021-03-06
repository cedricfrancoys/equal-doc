## Use eQual with external libs

### How to integrate external libraries ?


eQual natively support the [PSR-4 standard](https://www.php-fig.org/psr/psr-4/)

You can use composer to add to the `/vendor` folder located in the root folder of eQual installation.

Libraries that follow the PSR-4 standard can be loaded with a simple `use` statement. 
For libraries that do not provide such support, simply use 'require' or 'include' as stated in the documentation of the library.


### Examples

Here are a few examples : 

#### Twig - template engine

```bash
composer require "twig/twig:^2.0"
```

```
<?php
use Twig\Environment;
use Twig\Loader\FilesystemLoader;
```

#### PHPOffice - MS Office compatible documents generation library

```bash
composer require phpoffice/phpspreadsheet
```

```php
<?php
use PhpOffice\PhpSpreadsheet\Spreadsheet;
use PhpOffice\PhpSpreadsheet\IOFactory;
```

#### DOMPDF - HTML to PDF converter

```bash
composer require dompdf/dompdf
```

```php
<?php
use Dompdf\Dompdf;
use Dompdf\Options;
```


For libraries that do not support autoloading, it is still a good practive to store them in the `/vendor` folder, and then including them manually by using `include` or `require`:
```php
<?php
require_once '../vendor/swiftmailer/swiftmailer/lib/swift_required.php';
```

However, once included, you can still take advantage of the `use` statement in order to make the syntax smoother and not to bother with namespaces.

```php
<?php
require_once '../vendor/swiftmailer/swiftmailer/lib/swift_required.php';

use \Swift_SmtpTransport as Swift_SmtpTransport;
use \Swift_Message as Swift_Message;
use \Swift_Mailer as Swift_Mailer;
```



