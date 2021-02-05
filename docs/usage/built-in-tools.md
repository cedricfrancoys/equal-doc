# Utilities and stand-alone scripts

Some tools are included in the core to ease management and development of new packages.


Utility scripts allow to achieve some tasks that are not handled by the core, for instance checking the consistency of a new package.

Those act like some sort of plugins, written in PHP and located in the folder: `data/utils/`


Here is a complete list of the scripts that come out-of-the-box.


## Installation & Config utilities


## Package utilities



### Test package consistency

Consistency checks between DB and class as well as syntax validation for
classes (PHP), views (HTML) and translation files (json)


```bash
equal.run --do=test_package-consistency --package=mypackage
```

#### Run package test cases

```bash
equal.run --do=test_package --package=mypackage
```


### Init a Package
Creates a compatible database based on a SQL schema.

```bash
equal.run --do=init_package --package=mypackage
```





## Rights management utilities


Should be located at the root of eQual (where *run.php* is)

### Grant DB rights

Available rights: "create", "read", "update", "delete", "manage"

You can grant only one right for one entity at a time

```bash
equal.run --do=group_grant --group=2 --right=read --entity="mypackage\MyObject"
```







