---
title: "GHGemissions"
author: "julia"
date: "2022/4/22"
output: html_document
---

```{r}
library(readr)
library(tidyverse)
GHGemissions <-  read_csv("https://raw.githubusercontent.com/hereisjulia/R_Visualization/85557bc192912f146f17befda9ec756a094b70d8/Tasks/world%20GHG%20emissions/ghg-emissions_countries.csv",col_types = cols(`1990` = col_double()))

```
#資料介紹
這是世界各國溫室氣體(1990-2018)的排放量
我希望可以呈現各國排放的年度變化量以及占比：
>因此先行考量使用geom_area

在資料處理的部分，希望可以將資料分成：
美國、中國、歐洲、印度、其他

```{r}
aes(x=var1, y=var2, other=var3)
```

would mean that your data frame consists of columns: var1, var2, and var3 unless the varX is a vector value, such as say `aes(x=1992)` or `aes(x=c(1991,1992,1993))`. In that case, data frame does not have a column of value `1992` or `c(1991, 1992, 1993)`.


At the beginning of the course, we learn to use ggplot without a data frame, like:
```{r}
library(ggplot2)
ggplot()+
  geom_point(
    aes(
      x=c(1991, 1992, 1993),
      y=c(90.3, 92.4, 88.6)
    )
  )
```

This is to equip you with the concept of aesthetics mapping to the geometries on the plot. 

In practice, the mapping source values are provided by a data frame. If you have:
```{r}
    aes(
      x=c(1991, 1992, 1993),
      y=c(90.3, 92.4, 88.6)
    )
```

You can construct a data frame as
```{r}
myData = data.frame(
      x=c(1991, 1992, 1993),
      y=c(90.3, 92.4, 88.6)
)
```

and then

```{r}
ggplot(
  myData
)+
  geom_point(
    aes(
      x=x,
      y=y
    )
  )
```

As you can see, I simply replace `aes` with `data.frame` to construct the correponding data frame.

However, in practice the data frame comes from some data set which does not necessarily have columns named `x` and `y` respectively. It might look like:
```{r}
myData = data.frame(
      year=c(1991, 1992, 1993),
      emission=c(90.3, 92.4, 88.6)
)
```
In this case, we change the `aes` mapping source from `x` `y` to its corresponding column names as

```{r}
ggplot(
  myData
)+
  geom_point(
    aes(
      x=year,
      y=emission
    )
  )
```

Here inside `aes(...)` there are pairs of `LHS=RHS` expression. `LHS` refers to the aesthetic element that a geometry needs to be well defined for graphing. For example, point geometry requires the knowledge of its coordinate, i.e. `x` and `y`. So `x` and `y` are the necessary geometry aesthetics to be places on the `LHS`. It is fixed symbol that has nothing to do with your data frame column names. So point geometry always has `aes(x=..., y=...)`.

The `RHS` refers to the source of values that defines possible `LHS` aesthetic values. `RHS` usually are the column names of the data frame. 

In your case, you should start with a template as
```{r}
ggplot(
  data=GHGemissions
) + 
  geom_point(
    aes(
      x=,
      y=
    )
  )
```

The try to figure out which column of GHGemissions will supply the values of `x` and which will supply the values of `y`.

***

Unfortunately, in your case, there is no such columns since the value you want for `x` is the column name and `y` is the cell values. This kind of data frame in contrast with the data frame you wish for (the one with a column of Year and a column of emissionLevel) is call a wide form data frame (the one you wish for is a long form data frame).

This requires data manipulate to pivot a wide form data frame into a long form data frame. This is the job of `pivot_longer` (in contrast there is `pivot_wider` to do the reverse job).
```{r}
library(tidyverse)
pivot_longer(
  GHGemissions,
  cols= -c(1:2), # which columns in the current wide form data frame needs to pivot (here I use excluding columns 1 to 2 expression)
  names_to = "year", # once pivoted how would you call the source column names
  values_to = "emissionLevel" # once pivoted how would you call the cell values
) -> GHGemissions_long
```

Once you obtain the long form data frame, you can:

```{r}
ggplot(
  data=GHGemissions_long
) + 
  geom_point(
    aes(
      x=year,
      y=emissionLevel,
      color=`Country/Region`
    )
  ) +
  theme(
    legend.position = "none"
  )
```



