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

# Tidying up pandas? 

As an academic, often *the* go to  _lingua franca_ for data science might not always be python but R. This is especially so if you’re coming from a Computational Biology/Bioinformatics/Systems Biology or the Statistics Department. 

Likelihood will be you’ll be hooked on the famous `tidyverse` meta-package, which includes `dplyr` (previously ply(e)r), `lubridate` (time-series) and `tidyr` for example. 

> PS. As I wrote this article I realised it isn’t just `tidyverse`, but the whole R ecosystem which I’ve come to love whist doing metagenomics and computational biology in general. 

Nowadays, take no more than five steps you’ll see how much *python* and `pandas`  is used. For the uninitiated, `pandas` is _the_ data frame module for python, several other packages like [datatable](https://datatable.readthedocs.io/en/latest/using-datatable.html) which is heavily inspired by R’s own [datatable]([Introduction to data.table](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.html) library exists as well. 

Additionally, something which caught me off guard was SQL, postgreSQL and it’s dialects eg. Redshift and [KSQL](https://www.confluent.io/product/ksql/) for Kafka. 

In his talk, [Hadley Wickham](https://youtu.be/dWjSYqI7Vog?t=2m7s),  mentioned what we really need for table manipulation are just a handful of functions. 

* filter
* select
* arrange
* mutate
* group_by
* summarise
* merge

Although, I would argue you need just a bit more, for example knowing R’s family of `apply` functions will help you go a long way. Or a couple of summary statistics functions like `summary` or  `str` , although nowadays I use  `skimr`’s `skim`  a lot. 


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

google’s Facets (see image below works kind of like this as well)

![Facets](https://i.imgur.com/F7yQLnz.png)

Thus I’ve decided to write this post where I’ll try my best to demonstrate 1-to-1 mappings of the `tidyverse` vocabulary with `pandas` methods.

We will be using the famous. [Iris flower datasets](https://en.wikipedia.org/wiki/Iris_flower_data_set). 

```python
import seaborn as sns
iris = sns.load_data("iris")  # imports the dataset as a pandas DataFrame as compared to sklearn's datasets which are numpy arrays
```

```r
# iris is already loaded into the environment by default in R
iris
```

The first thing I usually do when I import a table is to run the `str` function on the table

```r
str(iris)
# use skimr
skim(iris)
```

```python
iris.info(null_counts=True)  
# if the number of rows are too much, pandas will not do the count, so I have to forcibly set `null_counts` to `True`.
```

But recently I’ve been really seduced by the `skimr` package in R. 

```r
library(skimr)  # from ROpenSci
iris %>% skim
```

## Filter
The closest method similar to R’s `filter` is  `pd.query`. 

```r
cutoff = 30
iris %>% 
    filter(sepal.width > cutoff)
```

There’s two ways to do this in python. The first is probably what you’ll find most python users using

```python
cutoff = 30
iris[iris.sepal_width > cutoff]
```

However, `pd.DataFrame.query()` maps more closely with `dplyr::filter()` 
. 
```python
iris. \
    query("sepal_width > @cutoff”)  # this is using a SQL like language
```

> One downside of using this is linters like pep8 and flake8 will complain the the `cutoff` variable although declared is not used as they are unable to recognise the use of `cutoff` inside the query quoted string. 

Surprisingly, filter makes a return in pySpark. :) 

```python
type(flights)
pyspark.sql.dataframe.DataFrame
```

```python
# filters flights which are > 1000 miles long
flights.filter('distance > 1000')
```

## Select 
this is reminiscent of SQL’s `select`  keyword which allows you to choose columns. 
```r
iris %>% 
    select(sepal.width, sepal.length)
```

```python
iris \
    .loc[:5, [["sepal_width", "sepal_length"]]]  # selects the 1st 5 rows 
```

Initially I chose the  `df[['col1', 'col2']]`  pattern. But realised we cannot do slices of the columns similar to `select`.  

```r
iris %>% 
    select(Sepal.Length:Petal.Width)
```

```python
iris. \
    loc[:, "sepal_length":"petal_width"]
```


A thing to note about the `loc` method is that it could return a series instead of a DataFrame when the selection is just one row

```
iris.loc[1, :] #selects the first row and returns a pandas.Series
```

But the really awesome thing about `select`, function its ability to /unselect/ columns which is missing in the `loc` method. 

```r
df %>% select(-col1) 
```

You have to use the `.drop()` method. 

```python
df.drop(columns=["col1"])
```

> Note i had to add the param `columns` because drop can not only be used to drop columns, the method can also drop rows based on their index. 

Like `filter`,  `select` is also used in pySpark! 

```python
df.select("xyz").show() # shows the column xyz of the spark dataframe.

# alternative 
df.select(df.xyz)
```

## Arrange
The arrange function lets one sort the table by a particular column 

```r
df %>% arrange(col1, descreasing=TRUE)
```

```python
df \
    .sort_values(by="col1", ascending=False)  # everything is reversed in python fml. 
```

## Mutate

`dplyr`’s `mutate` was really an upgrade from R’s `apply`. 
> **NOTE**: Other applies which is useful in R for example includes `mapply` and `lapply`


```r
df %>% mutate(
    new = something / col2, 
    newcol = col+1
)
```

```python 
iris.assign(
    new = iris.sepal_width / iris.sepal, 
    newcol = lambda x: x["col"] + 1
)
```


`tidyverse`’s `mutate` function by default takes the whole column and does vectorised operations on it. If you want to apply the function row by row, you’ll have to couple `rowwise` with `mutate`.

```R
# my_function does not take vectorised input of the entire row

# this will fail
iris %>% 
    mutate(new_column = my_function(sepal.width, sepal.length))

# this will force mutate to be applied row by row
iris %>% 
    rowwise %>%
    mutate(new_column = my_function(sepal.width, sepal.length))
```

To achieve the same using the `.assign` method you can nest an `apply`inside the function. 

```python
def do_something_string(col):
    #set_trace()
    if re.search(r".*(osa)$", col):
        value = "is_setosa"
    else:
        value = "not_setosa"
    return value

def dosomething(df):
    return df.species.apply(do_something_string)

iris = iris.assign(transformed_species = lambda df: dosomething(df))
iris
```

If you're lazy, you could just chain two anoymous functions together

```python
iris = iris.assign(transformed_species = 
                   lambda df: df["species"] \
                       .apply(
                           lambda col: "is_setosa" if re.search(r".*(osa)$", col) else "not_setosa")
                  )
````

However, this is slow and assign does **not** allow you carry out *row wise* computes. 

One method is to iterate, to fall back on `pd.DataFrame.iterrows` .

> There’s also `iteritems()` for `pd.Series`
> 
In pySpark, you can split the whole table into partitions. 

```python
dd.from_pandas(my_df,npartitions=nCores).\
   map_partitions(
      lambda df : df.apply(
         lambda x : nearest_street(x.lat,x.lon),axis=1)).\
   compute(get=get)
# imports at the end
```


## Apply
From R’s `apply` help docs:

```
apply(X, MARGIN, FUN, ...)
```

Where the value of `MARGIN` takes either `1` or `2` for (rows, columns), ie. if you want to apply to each row, you’ll set the axis as `0`. 

However, in pandas axis refers to what values (index i or columns j) will be used for the applied functions input parameter’s index. 

be using the  `0` refers to the DataFrame’s index and axis `1` refers to the columns. Which is still in line 

![Imgur](https://i.imgur.com/uNOGXVT.png)

So if you wanted to carry out row wise operations you could  set axis to 0
```
df %>% 
    apply(0, function(row){
        ...
        do some compute
        ...
})
```

> Rarely do that now since `plyr` and later `dplyr.` 

However in there no `plyr` in pandas. So we have to go back to using apply if you want row wise operations, however, the axis now is 1 not 0. I initially found this very confusing. The reason is because the *row* is a really just a `pandas.Series` whose index is the parent  pandas.DataFame’s columns. Thus in this the axis is referring to which axis to set as the index. 

```python
iris.apply(lambda row: do_something(row), axis=1)
```

Interesting pattern which I do not use in R, is to use apply on columns, in this case pandas.Series objects

```python
iris.sepal_width.apply(lambda x: x**2)

#  if you want a fancy progress bar, you could use the tqdm function
iris.sepal_width.apply_progress(lambda x: x**2) 

# If u need parallel apply
# this works with dask underneath 
import swifter
iris.sepal_width.swifter.apply(lambda x : x**2) 
```

In R, one of the common idioms, which i keep going back to for a parallel version of `groupby` is as follows. 

```r
unique_list %>% 
    lapply(function(x){ 
        ...
        df %>% filter(col == x) %>%
          do_something() # do something to the subset
          ...
}) %>% do.call(rbind,.)
```

If you want a parallel version you’ll just have to change the `lapply` to `mclapply`.

```r
ncores = 10  # the number of cores
unique_list %>% 
    mclapply(function(x){ 
        ...
        df %>% filter(col == x) %>%
          do_something() # do something to the subset
          ...
}, mc.cores=ncores) %>% do.call(rbind,.)
```

Separately, I could try using `pd.iterrows`,  which is similar to the `rowwise` function coupled with mutate. 

Additionally, there’s `mclapply` from the `parallel` /`snow` library in R. To achieve the same, what we can use the `dask`, or a higher level wrapper from the `swiftapply` library. 

```python
# you can easily vectorise the example using by adding the `swift` method before `.apply`
series.swift.apply()
```

## Group by
the `group_by` option is very useful and the equivalent in pandas is the `.groupby` method which returns a grouped DataFrame

> In Tidyverse there’s the `ungroup` function to ungroup grouped DataFrames, in order to achieve the same, there does not exists a1-to-1 mappable function. 
> 
> One way is to complete the `groupby` -> `apply` (two-step process) and feeding apply with an identity function `apply(lambda x: x)`. Which is an identity function. 


## Summarise
In pandas the equivalent of the `summarise` function is `aggregate`  abbreviated as the `agg` functions.

```
df %>% 
    group_by(col) %>% summarise(my_new_column = do_something(some_col))
```

However when you run the `agg` function you 

Not that you can actually chain `apply` or `agg` with `groupby`, in fact the closer function to R’s `summarise` would be agg

## Join
Natively, R supports the `merge` function and similarly in Pandas there’s the `pd.merge` function. 

Along side the other `join` functions: `left_join`, `right_join`, `inner_join` and `anti_join`


## Inplace

In R there’s the compound assignment pipe-operator  `%<>%`, which is similar to the `inplace=True` argument in some pandas functions *but not all*.  :(

### Debugging

In R, we have the `browser()` function. 

```R
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
from IPython.core.debugger import set_trace()

(
    iris
        .groupby("species")
        .apply(lambda groupedDF: set_trace())
)
```

With this 

Last but not least if you really need to use some R function you could always rely on the `rpy2` package


```python
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

Why you would want to 
```python
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
# dummy function which prints `kwargs`
def some_function (**kwargs): print(kwargs)

some_function(first=1, second=2)
```

The previous two cases are not exclusive, you could actually ~***mix***~ them together. Ie. have named signatures as well as a  `**kwargs`

```python
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


The output will be
```
{'useless_value': 'wesley'}
3
```

It allows a python function to accept as many function signatures as you supply it. Those which are already defined during the declaration of the function would be directly used. And those which do not appear within them can be accessed from kwargs.

By putting the `**kwargs` as an argument in the inner function, you’re basically unwrapping the 

```python
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

So far what we’ve seen is unique to python.  
￼
