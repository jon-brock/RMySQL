Code for Using Data in MySQL
================

  - [*Loading Required Packages*](#loading-required-packages)
  - [*Connecting to the MySQL Schema
    “Sakila”*](#connecting-to-the-mysql-schema-sakila)
  - [*Looking Inside the Sakila
    Schema*](#looking-inside-the-sakila-schema)
  - [*Pulling Data Out from the Sakila
    Schema*](#pulling-data-out-from-the-sakila-schema)
  - [*Pulling All Tables into the R Global
    Environment*](#pulling-all-tables-into-the-r-global-environment)

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

The `sakila` schema is a sample database that contains information on
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
qry_address_tbl <- dbSendQuery(sakila_db, "SELECT * FROM address")
```

2.  Retrieve the data from that R object using the `dbFetch()` function
    and assign it to a dataframe.

<!-- end list -->

``` r
address_df <- dbFetch(qry_address_tbl, n = -1)
```

If you prefer to work with data that is stored as a `tibble` then you
can convert the resulting `data.frame` using the following code:

*Note: you may need to re-parse your variables into more appropriate
data types if you convert to a tibble*

``` r
address_tb <- as_tibble(address_df)

#Remove the MySQLResult, as well as the superfluous data frame after conversion to a tibble
rm(qry_address_tbl, address_df)
```

*Note: Our results are too wide to show in a coherent manner, so we can
print out a sample of `20` street addresses instead.*

``` r
pull(address_tb, address)
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

-----

##### *Pulling All Tables into the R Global Environment*

As with most things in life there are multiple ways of doing things. We
can work with our data using two options:

1.  We can write our queries in advance and use R to pull our results
    into a data frame. (See process above)  
2.  We can pull all the tables from our designated MySQL server into R
    and manipulate/analyze them further.

Remember that we have the following 23 tables stored in our `sakila`
schema:

``` r
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

Additionally, you can see how the tables are all related to one another
in the image below. Knowing the relationships between tables is critical
to successfully querying the results that you need; some information we
will need, and some we will not.

![table\_relationships](https://dev.mysql.com/doc/sakila/en/images/sakila-schema.png)  
*([Click here for table-specific
information](https://dev.mysql.com/doc/sakila/en/sakila-structure-tables.html))*

There is, perhaps, an easier and simpler way of coding the data imports
and conversions for all of the `sakila` tables. However, at this time,
we can press onward with a bit of redundancy. The following code chunk
repeats what we did earlier:

1.  sends the query to MySQL  
2.  fetches the query results and assigns them to an object (data
    frame)  
3.  converts the corresponding data frame to a tibble  
4.  removes the superfluous objects from the working environment

<!-- end list -->

``` r
 #Actor Table
qry_actor_tbl <- dbSendQuery(sakila_db, "SELECT * FROM actor")
    actor_df <- dbFetch(qry_actor_tbl, n = -1)
        actor_tb <- as_tibble(actor_df)
            rm(qry_actor_tbl, actor_df)

#Actor_Info Table
qry_actor_info_tbl <- dbSendQuery(sakila_db, "SELECT * FROM actor_info")
    actor_info_df <- dbFetch(qry_actor_info_tbl, n = -1)
        actor_info_tb <- as_tibble(actor_info_df)
            rm(qry_actor_info_tbl, actor_info_df)

#Category Table
qry_category_tbl <- dbSendQuery(sakila_db, "SELECT * FROM category")
    category_df <- dbFetch(qry_category_tbl, n = -1)
        category_tb <- as_tibble(category_df)
            rm(qry_category_tbl, category_df)

#City Table            
qry_city_tbl <- dbSendQuery(sakila_db, "SELECT * FROM city")
    city_df <- dbFetch(qry_city_tbl, n = -1)
        city_tb <- as_tibble(city_df)
            rm(qry_city_tbl, city_df)

#Country Table            
qry_country_tbl <- dbSendQuery(sakila_db, "SELECT * FROM country")
    country_df <- dbFetch(qry_country_tbl, n = -1)
        country_tb <- as_tibble(country_df)
            rm(qry_country_tbl, country_df)

#Customer Table
qry_customer_tbl <- dbSendQuery(sakila_db, "SELECT * FROM customer")
    customer_df <- dbFetch(qry_customer_tbl, n = -1)
        customer_tb <- as_tibble(customer_df)
            rm(qry_customer_tbl, customer_df)
            
#Customer_List Table
qry_customer_list_tbl <- dbSendQuery(sakila_db, "SELECT * FROM customer_list")
    customer_list_df <- dbFetch(qry_customer_list_tbl, n = -1)
        customer_list_tb <- as_tibble(customer_list_df)
            rm(qry_customer_list_tbl, customer_list_df)

#Film Table
qry_film_tbl <- dbSendQuery(sakila_db, "SELECT * FROM film")
    film_df <- dbFetch(qry_film_tbl, n = -1)
        film_tb <- as_tibble(film_df)
            rm(qry_film_tbl, film_df)
            
#Film_Actor Table
qry_film_actor_tbl <- dbSendQuery(sakila_db, "SELECT * FROM film_actor") 
    film_actor_df <- dbFetch(qry_film_actor_tbl, n = -1)
        film_actor_tb <- as_tibble(film_actor_df)
            rm(qry_film_actor_tbl, film_actor_df)
            
#Film_Category Table
qry_film_category_tbl <- dbSendQuery(sakila_db, "SELECT * FROM film_category")
    film_category_df <- dbFetch(qry_film_category_tbl, n = -1)
        film_category_tb <- as_tibble(film_category_df)
            rm(qry_film_category_tbl, film_category_df)
            
#Film_List Table
qry_film_list_tbl <- dbSendQuery(sakila_db, "SELECT * FROM film_list")
    film_list_df <- dbFetch(qry_film_list_tbl, n = -1)
        film_list_tb <- as_tibble(film_list_df)
            rm(qry_film_list_tbl, film_list_df)
            
#Film_Text Table
qry_film_text_tbl <- dbSendQuery(sakila_db, "SELECT * FROM film_text")
    film_text_df <- dbFetch(qry_film_text_tbl, n = -1)
        film_text_tb <- as_tibble(film_text_df)
            rm(qry_film_text_tbl, film_text_df)
            
#Inventory Table
qry_inventory_tbl <- dbSendQuery(sakila_db, "SELECT * FROM inventory")
    inventory_df <- dbFetch(qry_inventory_tbl, n = -1)
        inventory_tb <- as_tibble(inventory_df)
            rm(qry_inventory_tbl, inventory_df)
            
#Language Table
qry_language_tbl <- dbSendQuery(sakila_db, "SELECT * FROM language")
    language_df <- dbFetch(qry_language_tbl, n = -1)
        language_tb <- as_tibble(language_df)
            rm(qry_language_tbl, language_df)
            
#Nicer_Bit_Slower_Film_List Table
qry_nicer_but_slower_film_list_tbl <- dbSendQuery(sakila_db, "SELECT * FROM nicer_but_slower_film_list")
    nicer_but_slower_film_list_df <- dbFetch(qry_nicer_but_slower_film_list_tbl, n = -1)
        nicer_but_slower_film_list_tb <- as_tibble(nicer_but_slower_film_list_df)
            rm(qry_nicer_but_slower_film_list_tbl, nicer_but_slower_film_list_df)
            
#Payment Table
qry_payment_tbl <- dbSendQuery(sakila_db, "SELECT * FROM payment")
    payment_df <- dbFetch(qry_payment_tbl, n = -1)
        payment_tb <- as_tibble(payment_df)
            rm(qry_payment_tbl, payment_df)
            
#Rental Table
qry_rental_tbl <- dbSendQuery(sakila_db, "SELECT * FROM rental")
    rental_df <- dbFetch(qry_rental_tbl, n = -1)
        rental_tb <- as_tibble(rental_df)
            rm(qry_rental_tbl, rental_df)
            
#Sales_By_Film_Category Table
qry_sales_by_film_category_tbl <- dbSendQuery(sakila_db, "SELECT * FROM sales_by_film_category") 
    sales_by_film_category_df <- dbFetch(qry_sales_by_film_category_tbl, n = -1)
        sales_by_film_category_tb <- as_tibble(sales_by_film_category_df)
            rm(qry_sales_by_film_category_tbl, sales_by_film_category_df)
            
#Sales_By_Store Table
qry_sales_by_store_tbl <- dbSendQuery(sakila_db, "SELECT * FROM sales_by_store")
    sales_by_store_df <- dbFetch(qry_sales_by_store_tbl, n = -1)
        sales_by_store_tb <- as_tibble(sales_by_store_df)
            rm(qry_sales_by_store_tbl, sales_by_store_df)
            
#Staff Table
qry_staff_tbl <- dbSendQuery(sakila_db, "SELECT * FROM staff")
    staff_df <- dbFetch(qry_staff_tbl, n = -1)
        staff_tb <- as_tibble(staff_df)
            rm(qry_staff_tbl, staff_df)
            
#Staff_List Table
qry_staff_list_tbl <- dbSendQuery(sakila_db, "SELECT * FROM staff_list")
    staff_list_df <- dbFetch(qry_staff_list_tbl, n = -1)
        staff_list_tb <- as_tibble(staff_list_df)
            rm(qry_staff_list_tbl, staff_list_df)
            
#Store Table
qry_store_tbl <- dbSendQuery(sakila_db, "SELECT * FROM store")
    store_df <- dbFetch(qry_store_tbl, n = -1)
        store_tb <- as_tibble(store_df)
            rm(qry_store_tbl, store_df)
```

As a result, we can see (using `ls()`) that all of our tables have been
appropriately imported (as tibbles) into our working environment.

    ##  [1] "actor_info_tb"                 "actor_tb"                     
    ##  [3] "address_tb"                    "category_tb"                  
    ##  [5] "city_tb"                       "country_tb"                   
    ##  [7] "customer_list_tb"              "customer_tb"                  
    ##  [9] "film_actor_tb"                 "film_category_tb"             
    ## [11] "film_list_tb"                  "film_tb"                      
    ## [13] "film_text_tb"                  "inventory_tb"                 
    ## [15] "language_tb"                   "nicer_but_slower_film_list_tb"
    ## [17] "payment_tb"                    "rental_tb"                    
    ## [19] "sakila_db"                     "sales_by_film_category_tb"    
    ## [21] "sales_by_store_tb"             "staff_list_tb"                
    ## [23] "staff_tb"                      "store_tb"

-----
