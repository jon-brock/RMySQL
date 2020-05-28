Code for Using Data in MySQL
================

##### *Loading Required Packages*

``` r
library(RMySQL)
library(tidyverse)
library(keyring)
```

The `keyring`
[package](https://www.r-bloggers.com/how-to-hide-a-password-in-r-with-the-keyring-package/)
is needed in order to maintain the security and privacy of our login
information. This is crucial because we do not want to share login
information when we distribute our code to others. We generate a key for
our password and use it by calling it with the `key_get()` function in
the `keyring` package. Coding this key is done [directly in the
console](https://www.infoworld.com/article/3320999/r-tip-keep-your-passwords-and-tokens-secure-with-the-keyring-package.html)
and not included in the final code.

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

-----

##### *Looking Inside the Sakila Schema*

``` r
 #List all of the tables
dbListTables(sakila_db)
```

    ##  [1] "actor"                      "actor_info"                
    ##  [3] "address"                    "category"                  
    ##  [5] "city"                       "country"                   
    ##  [7] "customer"                   "customer_list"             
    ##  [9] "film"                       "film_actor"                
    ## [11] "film_category"              "film_list"                 
    ## [13] "film_text"                  "inventory"                 
    ## [15] "language"                   "nicer_but_slower_film_list"
    ## [17] "payment"                    "rental"                    
    ## [19] "sales_by_film_category"     "sales_by_store"            
    ## [21] "staff"                      "staff_list"                
    ## [23] "store"

We can see from our results that the schema contains 23 distinct tables.
Some of these tables are strictly related to the cusomter, whereas the
rest of the tables are related to the films.

We can assess what variables (or columns) are within a specific table
using the following code:

``` r
dbListFields(sakila_db, "address")
```

    ## [1] "address_id"  "address"     "address2"    "district"    "city_id"    
    ## [6] "postal_code" "phone"       "location"    "last_update"

The `address` table contains nine variables, as seen above. Now that we
know how to connect to our MySQL database and see what tables and
variables lie within it it’s time to start exploring.

-----

##### *Pulling Data Out from the Sakila Schema*

Let’s start with something basic. I want to see all of the data
contained within the `address` table.

*Querying the data is a two-part process:*

1.  Assign a query to a designated R object.

<!-- end list -->

``` r
query_address_all <- dbSendQuery(sakila_db, 
                                        "SELECT *
                                         FROM address")
```

2.  Retrieve the data from that R object using the `fetch()` function
    and assign it to a dataframe.

<!-- end list -->

``` r
data_query_address_all <- fetch(query_address_all, n = -1)
```

*Note: Our results are too wide to show in a coherent manner, so we can
print out a sample of `20` street addresses instead.*

``` r
pull(data_query_address_all,address)
```

    ##  [1] "47 MySakila Drive"                  "28 MySQL Boulevard"                
    ##  [3] "23 Workhaven Lane"                  "1411 Lillydale Drive"              
    ##  [5] "1913 Hanoi Way"                     "1121 Loja Avenue"                  
    ##  [7] "692 Joliet Street"                  "1566 Inegl Manor"                  
    ##  [9] "53 Idfu Parkway"                    "1795 Santiago de Compostela Way"   
    ## [11] "900 Santiago de Compostela Parkway" "478 Joliet Way"                    
    ## [13] "613 Korolev Drive"                  "1531 Sal Drive"                    
    ## [15] "1542 Tarlac Parkway"                "808 Bhopal Manor"                  
    ## [17] "270 Amroha Parkway"                 "770 Bydgoszcz Avenue"              
    ## [19] "419 Iligan Lane"                    "360 Toulouse Parkway"              
    ##  [ reached getOption("max.print") -- omitted 583 entries ]
