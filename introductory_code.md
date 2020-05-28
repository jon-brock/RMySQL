Code for Using Data in MySQL
================

##### *Loading Required Packages*

``` r
library(RMySQL)
library(tidyverse)
library(keyring)
```

The `keyring` package is needed in order to maintain the security and
privacy of our login information. This is crucial because we do not want
to share login information when we distribute our code to others. We
generate a key for our password and use it by calling it with the
`key_get()` function in the `keyring` package. Coding this key is done
directly in the console and not included in the final code.

-----

##### *Connecting to the MySQL Schema “Sakila”*

``` r
sakila_db = dbConnect(MySQL(),
            user = "root",
            password = key_get("root_pw"),
            dbname = 'sakila',
            host = 'localhost')
```

The `Sakila` schema is a sample database that contains information on
movie rental customers, as well as information on the movies themselves.
You can find both the SQL schema documentation and zip file for download
[here](https://dev.mysql.com/doc/sakila/en/) and
[here](https://downloads.mysql.com/docs/sakila-db.zip), respectively.
