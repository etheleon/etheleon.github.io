---
layout: post
title: "Tidying Up Pandas"
excerpt: "Transitioning from Academia to Industry: Using Python DataFrames"
categories: articles
tags: [pandas, tidyverse, R, python]
author: wesley_goi
comments: true
share: true
modified: 2018-12-16
mathjax: true
image:
  feature: tidypandas.png
  thumb: tidypandas.png
  credit: Wesley GOI
  creditlink: http://etheleon.github.io
---

For those who use python's pandas model daily, the first thing you would notice is there are often more ways than one to do almost everything. 

The purpose of this article is to demonstrate how we can limit this by drawing inspiration from R's `dplyr` and `tidyverse` libraries

# Tidying up pandas? 

As an academic, often enough the go to  _lingua franca_ for data science is R. Especially if you’re coming from Computational Biology/Bioinformatics or Statistics. 

And likely you’ll be hooked on the famous `tidyverse` meta-package, which includes `dplyr` (previously `plyr` for ply(e)r), `lubridate` (time-series) and `tidyr`. 

> PS. As I am writing this article I realised it isn’t just `tidyverse`, but the whole R ecosystem which I’ve come to love whist doing metagenomics and computational biology in general. 

For the benefit of those who started from R, `pandas` is _the_ dataframe module for python, several other packages like [datatable](https://datatable.readthedocs.io/en/latest/using-datatable.html) exists and is is heavily inspired by R’s own [datatable](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.html). 

Now back to how tidyverse specifically dplyr organises dataframe manipulation. 

In his talk, [Hadley Wickham](https://youtu.be/dWjSYqI7Vog?t=2m7s), mentioned what we really need for table manipulation are just a handful of functions. 

* filter
* select
* arrange
* mutate
* group_by
* summarise
* merge

Although, I would argue you need just a bit more. For example, knowing R’s family of `apply` functions will help  tonnes. Or a couple of summary statistics functions like `summary` or  `str` , although nowadays I use  `skimr::skim` a lot. 


```r
skim(iris)

## Skim summary statistics
##  n obs: 150 
##  n variables: 5 
## 
## ── Variable type:factor ──────────────────────────────────────────────────────────────────────────────────────────────────
##  variable missing complete   n n_unique                       top_counts ordered
##   Species       0      150 150        3 set: 50, ver: 50, vir: 50, NA: 0   FALSE
## 
## ── Variable type:numeric ─────────────────────────────────────────────────────────────────────────────────────────────────
##      variable missing complete   n mean   sd  p0 p25  p50 p75 p100     hist
##  Petal.Length       0      150 150 3.76 1.77 1   1.6 4.35 5.1  6.9 ▇▁▁▂▅▅▃▁
##   Petal.Width       0      150 150 1.2  0.76 0.1 0.3 1.3  1.8  2.5 ▇▁▁▅▃▃▂▂
##  Sepal.Length       0      150 150 5.84 0.83 4.3 5.1 5.8  6.4  7.9 ▂▇▅▇▆▅▂▂
##   Sepal.Width       0      150 150 3.06 0.44 2   2.8 3    3.3  4.4 ▁▂▅▇▃▂▁▁
```

In fact, Google’s Facets behaves somewhat like this as well (see image below).

![Facets](https://i.imgur.com/F7yQLnz.png)

Thus, in this post I’ll try my best to demonstrate 1-to-1 mappings of the `tidyverse` vocabularies with `pandas` methods.

For demonstraiton, We will be using the famous [Iris flower dataset](https://en.wikipedia.org/wiki/Iris_flower_data_set). 

```python
# python

import seaborn as sns
iris = sns.load_data("iris")  
```

I've chosen to imports the iris data using seaborn rather than sklearn's datasets which are numpy arrays

The first thing I usually do when I import a table is to run the `str` function on the table

```r
# R (iris is already loaded by default)

str(iris)
```

```python
# python

iris.info(null_counts=True)  
# if the number of rows are too much, pandas will not do the count, 
# so I have to forcibly set `null_counts` to `True`.
```

## Filter
The closest method similar to R’s `filter` is  `pd.query`. 

```r
# R

cutoff = 30
iris %>% 
    filter(sepal.width > cutoff)
```

There’s two ways to do this in python. The first is probably what you’ll find most python users using

```python
# python

cutoff = 30
iris[iris.sepal_width > cutoff]
```

However, `pd.DataFrame.query()` maps more closely with `dplyr::filter()`. 

```python
# R

iris. \
    query("sepal_width > @cutoff”)  # this is using a SQL like language
```

> One downside of using this is linters which follows the `pep8` convention like `flake8` will complain about the `cutoff` variable not being used although it has already been declared. This is because the linters are unable to recognise the use of `cutoff` inside the query quoted string. 

Surprisingly, filter makes a return in pySpark. :) 

```python
# python (pyspark)

type(flights)
pyspark.sql.dataframe.DataFrame

# filters flights which are > 1000 miles long
flights.filter('distance > 1000')
```

## Select 

this is reminiscent of SQL’s `select` keyword which allows you to choose columns. 

```r
# R

iris %>% 
    select(sepal.width, sepal.length)
```

```python
# Python

iris \
    .loc[:5, [["sepal_width", "sepal_length"]]]  # selects the 1st 5 rows 
```

Initially, I thought the following `df[['col1', 'col2']]`  pattern would be a good map. But quickly realised we cannot do slices of the columns similar to `select`.  

```r
# R

iris %>% select(Sepal.Length:Petal.Width)
```

```python
# Python 

iris.loc[:, "sepal_length":"petal_width"]
```


A thing to note about the `loc` method is that it could return a series instead of a DataFrame when the selection is just one row. so you'll have to slice it in order to return a dataframe.

```
# Python

iris.loc[1, :]  # returns a Series
iris.loc[[1],:] # returns a dataframe
```

But the really awesome thing about `select`, function its ability to /unselect/ columns which is missing in the `loc` method. 

```r
# R

df %>% select(-col1) 
```

You have to use the `.drop()` method. 

```python
# Python

df.drop(columns=["col1"])
```

> Note I had to add the param `columns` because drop can not only be used to drop columns, the method can also drop rows based on their index. 

Like `filter`,  `select` is also used in pySpark! 

```python
# python (pySpark)

df.select("xyz").show() # shows the column xyz of the spark dataframe.

# alternative 
df.select(df.xyz)
```

## Arrange
The arrange function lets one sort the table by a particular column. 

```r
# R

df %>% arrange(col1, descreasing=TRUE)
```

```python
# Python

df.sort_values(by="col1", ascending=False)  # everything is reversed in python fml. 
```

## Mutate

`dplyr`’s `mutate` was really an upgrade from R’s `apply`. 

> **NOTE**: Other applies which is useful in R for example includes `mapply` and `lapply`


```r
# R

df %>% mutate(
    new = something / col2, 
    newcol = col+1
)
```

```python 
# Python

iris.assign(
    new = iris.sepal_width / iris.sepal, 
    newcol = lambda x: x["col"] + 1
)
```

`tidyverse`’s `mutate` function by default takes the whole column and does vectorised operations on it.
If you want to apply the function row by row, you’ll have to couple `rowwise` with `mutate`.

```R
# R

# my_function does not take vectorised input of the entire column

# this will fail
iris %>% 
    mutate(new_column = my_function(sepal.width, sepal.length))

# this will force mutate to be applied row by row
iris %>% 
    rowwise %>%
    mutate(new_column = my_function(sepal.width, sepal.length))
```

To achieve the same using the `.assign` method you can nest an `apply` inside the function. 

```python
# Python 

def do_something_string(col):
    #set_trace()
    if re.search(r".*(osa)$", col):
        value = "is_setosa"
    else:
        value = "not_setosa"
    return value

iris = iris.assign(
	transformed_species = lambda df: df["species"] \
		.apply(do_something_string)
	)
```

If you're lazy, you could just chain two anoymous functions together.

```python
# Python 

iris = iris.assign(
    transformed_species = lambda df: df.species.apply(do_something_string))
````

## Apply
From R’s `apply` help docs:

```
apply(X, MARGIN, FUN, ...)
```

Where the value of `MARGIN` takes either `1` or `2` for (rows, columns), ie. if you want to apply to each row, you’ll set the axis as `0`. 

However, in pandas axis refers to what values (index i or columns j) will be used for the applied functions input parameter’s index. 

be using the `0` refers to the DataFrame’s index and axis `1` refers to the columns.

![Imgur](https://i.imgur.com/uNOGXVT.png)

So if you wanted to carry out row wise operations you could set axis to 0.

```r
# R

df %>% 
    apply(0, function(row){
        ...
        do some compute
        ...
})
```

> Rarely do that now since `plyr` and later `dplyr.` 

However there is no `plyr` in pandas. So we have to go back to using apply if you want row-wise operations, however, the axis now is 1 not 0. I initially found this very confusing. The reason is because the *row* is a really just a `pandas.Series` whose index is the parent  pandas.DataFame’s columns. Thus in this the axis is referring to which axis to set as the index. 

```python
# python

iris.apply(lambda row: do_something(row), axis=1)
```

Interesting pattern which I do not use in R, is to use apply on columns, in this case `pandas.Series` objects.

```python
# python

iris.sepal_width.apply(lambda x: x**2)

#  if you want a fancy progress bar, you could use the tqdm function
iris.sepal_width.apply_progress(lambda x: x**2) 

# If u need parallel apply
# this works with dask underneath 
import swifter
iris.sepal_width.swifter.apply(lambda x : x**2) 
```

In R, one of the common idioms, which I keep going back to for a parallel version of `groupby` is as follows:

```r
# R

unique_list %>% 
    lapply(function(x){ 
        ...
        df %>% filter(col == x) %>%
          do_something() # do something to the subset
          ...
}) %>% do.call(rbind,.)
```

If you want a parallel version you’ll just have to change the `lapply` to `mclapply`.


Additionally, there’s `mclapply` from the `parallel` /`snow` library in R.

```r
# R

ncores = 10  # the number of cores
unique_list %>% 
    mclapply(function(x){ 
        ...
        df %>% filter(col == x) %>%
          do_something() # do something to the subset
          ...
}, mc.cores=ncores) %>% do.call(rbind,.)
```

Separately, in pySpark, you can split the whole table into partitions and do the manipulations in parallel.

```python
# Python (pyspark)

dd.from_pandas(my_df,npartitions=nCores).\
   map_partitions(
      lambda df : df.apply(
         lambda x : nearest_street(x.lat,x.lon),axis=1)).\
   compute(get=get)
# imports at the end
```

To achieve the same, what we can use the `dask`, or a higher level wrapper from the `swiftapply` library. 

```python
# Python

# you can easily vectorise the example using by adding the `swift` method before `.apply`
series.swift.apply()
```

## Group by

The `.groupby` method in pandas is equivalent to R function `dplyr::group_by` returning a `DataFrameGroupBy` object.

> In Tidyverse there’s the `ungroup` function to ungroup grouped DataFrames, in order to achieve the same, there does not exists a1-to-1 mappable function. 
> 
> One way is to complete the `groupby` -> `apply` (two-step process) and feeding apply with an identity function `apply(lambda x: x)`. Which is an identity function. 

## Summarise

In pandas the equivalent of the `summarise` function is `aggregate`  abbreviated as the `agg` function. And you will have to couple this with `groupby`, so it'll similar again a two step `groupby` -> `agg` transformation.

```r
# R

r_mt = mtcars %>% 
	mutate(model = rownames(mtcars)) %>%
	select(cyl, model, hp, drat) %>%
    filter(cyl < 8) %>%
    group_by(cyl) %>%
    summarise(
        hp_mean = mean(hp), 
        drat_mean = mean(drat),
        drat_std = sd(drat),
        diff = max(drat) - min(drat)
     ) %>% 
    arrange(drat_mean) %>%
    as.data.frame
```

The same series of transformation written in Python would follow:

```
# Python

def transform1(x):
    return max(x)-min(x)
        
def transform2(x):
    return max(x)+5

py_mt = (
mtcars.
    loc[:,["cyl", "model", "hp", "drat"]]. #select
    query("cyl < 8").                      #filter
    groupby("cyl").                        #group_by
    agg(                                   #summarise, agg is an abbreviation of aggregation
        {
            'hp':'mean', 
            'drat':['mean', 'std', transform1, transform2] # R wins... this sux for pandas
        }).
    sort_values(by=[("drat", "mean")])     #multindex sort (unique to pandas)
)
py_mt
```
   
```r
# R

df %>% 
    group_by(col) %>% 
    summarise(my_new_column = do_something(some_col))
```

## Join

Natively, R supports the `merge` function and similarly in Pandas there’s the `pd.merge` function. 

Along side the other `join` functions: `left_join`, `right_join`, `inner_join` and `anti_join`.


## Inplace

In R there’s the compound assignment pipe-operator  `%<>%`, which is similar to the `inplace=True` argument in some pandas functions *but not all*.  :(
Apparently Pandas is going to remove inplace altogether...


### Debugging

In R, we have the `browser()` function. 

```R
# R

unique(iris$species) %>%
    lapply(function(s){
        browser()
        iris %>% filter(species == s)
        ....
    })
```

It’ll let you *step* into the function which is extremely useful if you want to do some debugging. 

In Python, there’s the `set_trace`  function. 

```python 
# Python

from IPython.core.debugger import set_trace

(
    iris
        .groupby("species")
        .apply(lambda groupedDF: set_trace())
)
```

Last but not least if you really need to use some R function you could always rely on the `rpy2` package. For me I rely on this a lot for plotting. ggplot2 ftw!


```python
# python

import rpy2                #  imports the library
%load_ext rpy2.ipython     #  load the magic
```

> Sometimes there’s issues installing r packages using R. You can run 

`conda install -r r r-tidyverse r-ggplot`

There after you can always use R and Python interchangeably    in the same Jupyter notebook.

```r
%%R -i python_df -o transformed_df

transformed_df = python_df %>% 
    select(-some_columns) %>%
    mutate(newcol = somecol * 2)
```

> NOTE:  `%%R` is cell magic and `%R` is line magic. 

If you need outputs to be printed like a normal pandas DataFrame, you can you the single percent magic

```
%R some_dataFrame %>% skim
```

## Elipisis 
In R, one nifty trick you can do is to pass arguments to inner functions without ever having to define them in the outer function’s function signature.

```r
# R

#' Simple function which takes two parameters `one` and `two` and elipisis `...`, 
somefunction = function(one, two, ...){
   three =  one + two 
   sometwo = function(x, four){
        x + four
    }
    sometwo(three,  ...) # four exists within the elipisis 
}

# because of the elipisis, we can pass as many parameters as we we want. the extras will be stored in the elipisis
somefunction(one=2, two=3, four=5, name="wesley")
```

In python, `**kwargs` takes the place of `...`. Below is an explanation of how exactly it works. 

#### Explanation
Firstly, the double asterisks `**` is called *unpack* operator (it’s placed before a function signature eg. `kwargs` so together it’ll look like `**kwargs`).

> The convention is to let that variable be named `kwargs` (which stands for  **k**ey**w**orded arguments) but it could be named anything.

Most articles which describe the unpack operator will start off with **this** explanation: where dictionaries are used to pass functions their parameters.

```python
# Python

adictionary = {
    'first' : 1,
    'second': 2
}

def some_function(first, second):     return first + second

some_function(**adictionary)
# which gives 3
```

![unpacking](https://i.imgur.com/ggSP2dK.jpg)

But you could also twist this around and set `**kwargs` as a function signature. Doing this lets you key in an arbitrary number of function signatures when calling the function. 

The signature-value pairs are wrapped into a dictionary named `kwargs` which is accessible inside the function. 

```python
# Python

# dummy function which prints `kwargs`
def some_function (**kwargs): print(kwargs)

some_function(first=1, second=2)
```

The previous two cases are not exclusive, you could actually ~***mix***~ them together. Ie. have named signatures as well as a  `**kwargs`.

```python
# Python

adictionary = {
    'first' : 1,
    'second': 2,
    'useless_value' : "wesley"
}

def some_function(first, second, **kwargs):
    print(kwargs)
    return first + second

print(some_function(**adictionary))
```


The output will be: `{'useless_value': 'wesley'}`

It allows a python function to accept as many function signatures as you supply it. Those which are already defined during the declaration of the function would be directly used. And those which do not appear within them can be accessed from kwargs.

By putting the `**kwargs` as an argument in the inner function, you’re basically unwrapping the dictionary into the function params.

```python
# Python

def somefunction(one, two, **kwargs):
    print(f"outer function:\n\t{kwargs}")
    three = one + two
    def sometwo(x, four):
        print(f"inner function: \n\t{kwargs}")
        return x + four
    return sometwo(three, **kwargs)

somefunction(one=2, two=3, four=5, name=“wesley”)
```

```
outside function: 
    {“four”:5, “name”:”wesley”}
Inside
    
inside kwargs:
    {'name': 'jw'}
```

Lets now compare this with the original R elipsis

```r
# R
#' Simple function which takes two parameters `one` and `two` and elipisis `...`, 
somefunction = function(one, two, ...){
   three =  one + two 
   sometwo = function(x, four){
        x + four
    }
    sometwo(three,  ...) # four exists within the elipisis 
}

# because of the elipisis, we can pass as many parameters as we we want. the extras will be stored in the elipisis
somefunction(one=2, two=3, four=5, name="wesley")
```

## Conclusion

There's many ways to do thing in pandas more so than the tidyverse way, and I wish this was clearer.

Additionally, something which caught me off guard aftering coming to Honestbee was the amount of SQL I need.

For example postgreSQL to query RDS and it’s dialect for querying Redshift, [KSQL](https://www.confluent.io/product/ksql/) for querying data streams via Kafka and Athena's query language build on top of presto DB for querying S3, where most of the data use to exist in parquet files.

The shows one big deviation from academia where data in a company is usually stored in a database / datalake / datastream whereas in academia its usually just one big flat data file. 

We've come to the ending of this attempt at mapping tidyverse vocabularies to pandas, hope you've found this informative and useful! See you guys soon!
