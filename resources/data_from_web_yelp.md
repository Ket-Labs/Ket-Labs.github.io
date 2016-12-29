---
layout: page
title: "Getting Data from Web - Yelp"
permalink: /resources/data_from_web_yelp/
---

## Load libraries


```r
library(httr); library(httpuv); library(jsonlite); library(dplyr)
```

## Load keys and other autorizaiton information



Inside this file you will have something like this: (example with arbitrary values - you should obtain working values for [yelp API](https://www.yelp.com/developers/api_console) and specify them inside the file)


```r
consumerKey_Yelp = "alsdfjl34l23kj"
consumerSecret_Yelp = "lsdfkjslkj3"
token_Yelp = "lsadkfjwejf"
token_secret_Yelp = "alsdkfjasldkfj"
```

## Getting data from Yelp

Bellow we provide a minimum working example. More information you can find here:

* General idea about the data you can get from Yelp [here](https://www.yelp.com/developers/documentation/v2/overview)
* Create [Yelp account and create an API key](https://www.yelp.com/developers/api_console)
* [Authentication](https://www.yelp.com/developers/documentation/v2/authentication)
* [All parameters available in Yelp search API](https://www.yelp.com/developers/documentation/v2/search_api).  


```r
# authorization
myapp = oauth_app("YELP", key=consumerKey_Yelp, secret=consumerSecret_Yelp)
sig=sign_oauth1.0(myapp, token=token_Yelp,token_secret=token_secret_Yelp)

# Formulate a request string for 12 shops in NYC
limit <- 12;
location <- "NYC"
term <- "shop"
yelpurl <- paste0("http://api.yelp.com/v2/search/,",
                  "?limit=", limit, 
                  "&location=", location,
                  "&term=", term)

loaded_data <- GET(yelpurl, sig) # use verbose() inside GET for more details
```

## Reformat the data to a convinient format


```r
df <- loaded_data %>% # use names(loaded_data) to see what this object contains
  content() %>%   # extract only content from loaded_data objects
  jsonlite::toJSON() %>%  
  jsonlite::fromJSON() %>%
  as.data.frame()

# Explore data
# View(df)
```

### List to vector

Some variables in data frame are represented as a list. Let us convert one of them to vector


```r
str(df$businesses.categories)
```

```
## List of 12
##  $ : chr [1, 1:2, 1] "Women's Clothing" "womenscloth"
##  $ : chr [1:3, 1:2, 1] "Home Decor" "Arts & Crafts" "Antiques" "homedecor" ...
##  $ : chr [1, 1:2, 1] "Shopping Centers" "shoppingcenters"
##  $ : chr [1:3, 1:2, 1] "Shopping Centers" "Food Court" "Cultural Center" "shoppingcenters" ...
##  $ : chr [1:3, 1:2, 1] "Men's Clothing" "Formal Wear" "Hats" "menscloth" ...
##  $ : chr [1:3, 1:2, 1] "Jewelry" "Used, Vintage & Consignment" "Accessories" "jewelry" ...
##  $ : chr [1:3, 1:2, 1] "Gift Shops" "Specialty Schools" "Accessories" "giftshops" ...
##  $ : chr [1:3, 1:2, 1] "Women's Clothing" "Men's Clothing" "Children's Clothing" "womenscloth" ...
##  $ : chr [1:3, 1:2, 1] "Women's Clothing" "Accessories" "Lingerie" "womenscloth" ...
##  $ : chr [1, 1:2, 1] "Cosmetics & Beauty Supply" "cosmetics"
##  $ : chr [1, 1:2, 1] "Fashion" "fashion"
##  $ : chr [1:2, 1:2, 1] "Used, Vintage & Consignment" "Thrift Stores" "vintage" "thrift_stores"
```

```r
library(stringr)

df$businesses.categories <- df$businesses.categories %>%
  lapply(as.vector) %>%
  lapply(tolower) %>%
  lapply(str_replace_all, " ", "") %>%
  lapply(unique) %>% # removing repetitions
  sapply(str_c, collapse =", ")

df$businesses.categories
```

```
##  [1] "women'sclothing, womenscloth"                                                          
##  [2] "homedecor, arts&crafts, antiques, artsandcrafts"                                       
##  [3] "shoppingcenters"                                                                       
##  [4] "shoppingcenters, foodcourt, culturalcenter, food_court"                                
##  [5] "men'sclothing, formalwear, hats, menscloth"                                            
##  [6] "jewelry, used,vintage&consignment, accessories, vintage"                               
##  [7] "giftshops, specialtyschools, accessories"                                              
##  [8] "women'sclothing, men'sclothing, children'sclothing, womenscloth, menscloth, childcloth"
##  [9] "women'sclothing, accessories, lingerie, womenscloth"                                   
## [10] "cosmetics&beautysupply, cosmetics"                                                     
## [11] "fashion"                                                                               
## [12] "used,vintage&consignment, thriftstores, vintage, thrift_stores"
```

Of course, some subsequent cleaning will be needed. For example "and" and "&" should be treated equally, etc.
