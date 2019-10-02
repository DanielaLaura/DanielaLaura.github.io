---
published: true
title: Minimum subset of Pandas
date: 2019-07-07
categories:
  - pandas
  - data analysis
  - Python
---  
## Minimum subset of Pandasm 



The whole point of a data analysis library should be to provide you with the tools so that you can focus on the data analysis. While Pandas does provide you with the right tools, it doesn’t do so in a way that allows you to focus on the analysis. 
Instead, users are forced to tread through the complex and overabundant syntax.


There is a small subset of the library that is sufficient to accomplish nearly everything that it has to offer. It allows you to focus on doing data analysis and not the syntax.

With this minimum subset of Pandas:

- Your code will be simple, explicit, straightforward, and boring
- You will choose one obvious way to accomplish a task
- You will use this obvious way every single time
- You won’t have to retain as many commands in working memory
- Your code will be easier to understand by others and by you

<!--more-->

* * *
### Multitude Stack Overflow Answers
It is not uncommon to search for Pandas answers on Stack Overflow only to be met with several competing and varied results for common tasks. Treading through this deluge of information makes it difficult for those wanting to know the one idiomatic way to complete a task that they can commit to memory.

No Tricks
Eliminating much of the library will come with some (good) limitations. Knowing many obscure Pandas tricks might impress your friends, but it can lead to long lines of code that are difficult to understand and may be harder to debug.

###  Specific Pandas Examples
We will now cover a series of specific examples within Pandas where multiple approaches exist to complete a task. I will compare and contrast the different approaches and give guidance on which one I prefer. Listed below are the topics I cover.

- Selecting a single column of data
- Converting object column to datatime
- The deprecated ix indexer
- Selection with at and iat
- read_csv vs read_table duplication
- isna vs isnull and notna vs notnull
- groupby aggregation
- Handling a MultiIndex
- The similarity between melt and stack
- The similarity between pivot and unstack


The concrete examples were all derived by the following principle: If a method does not provide any additional functionality over another method (i.e. its functionality is a subset of another) then it shouldn’t be used.
Methods should only be considered if they have some additional, unique functionality.

* * *

### Selecting a Single Column of Data

Selecting a single column of data from a Pandas DataFrame is just about the simplest task you can do and unfortunately, it is here where we first encounter the multiple-choice option that Pandas presents to its users.
You may select a single column as a Series with either the brackets or dot notation. Let’s read in a small, trivial DataFrame and select a column using both methods.

    import pandas as pd
    df=pd.read_csv('play_evaluation.csv', sep=';')
	df=df.replace('2015-11-31', np.nan).dropna()
    df.head()

![](/images/1.png)

    df['Platform']

![](/images/2.png)

Selection with dot notation Alternatively, you may select a single column with dot notation. Simply, place the name of the column after the dot operator. The output is the exact same as above.
   df.Platform

![](/images/2.png)

### Issues with the dot notation
There are three issues with using dot notation. It doesn’t work in the following situations:

- When there are spaces in the column name
- When the column name is a variable
- The brackets are a strict superset of the dot notation in terms of functionality for selecting a single column. There are three cases which are not handled by the dot notation. Many tutorials make use of the dot notation to select a single column of data.
Why is this done when the brackets seem to be clearly superior? It might be because the official documentation contains plenty of examples that use it. It also uses three fewer characters which entice the very laziest amongst us.

The dot notation provides no additional functionality over the brackets and does not work in all situations. Therefore, I never use it. Its single advantage is three fewer keystrokes.

I suggest using only the brackets for selecting a single column of data. Having just a single approach to this very common task will make your Pandas code much more consistent.

* * *

### Converting object column to datatime

Though pd.to_datetime is very good at parsing dates normally without any format specified, when you actually specify a format, it has to match exactly. 
Any potential problems you might encouter is white space, which can be solved by adding .str.strip() to remove the extra white space before converting. 
Also, in case of some bad data, add parameter errors='coerce', which converts it to NaT (NaN for datetime)

    df['Date'] = pd.to_datetime(df['Date'], format='%Y/%m/%d')
    df.info()

![](/images/3.png)

### The deprecated ix indexer
To make selections explicit, the loc and iloc indexers were made available. The loc indexer selects only by label while the iloc indexer selects only by integer location. Although the ix indexer was versatile, it has been deprecated in favor of the loc and iloc indexers.

### Selection with at and iat
Two additional indexers, at and iat, exist that select a single cell of a DataFrame. These provide a slight performance advantage over their analogous loc and iloc indexers. But, they introduce the additional burden of having to remember what they do. 
Also, for most data analyses, the increase in performance isn’t useful unless it’s being done at scale. And if performance truly is an issue, then taking your data out of a DataFrame and into a NumPy array will give you a large performance gain.

### Performance comparison iloc vs iat vs NumPy
Here we create a NumPy array with 100k rows and 5 columns containing random data. We then create a DataFrame out of it and make the selections.
     import numpy as np
     a = np.random.rand(10 ** 5, 5)
     df1 = pd.DataFrame(a)
     row = 50000
     col = 3

     %timeit df1.iloc[row, col]
     10.3 µs ± 680 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
     %timeit df1.iat[row, col]
     6.83 µs ± 259 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
     %timeit a[row, col]
     203 ns ± 11.1 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)

While iat is a little less than twice as fast asiloc, selection with a NumPy array is about 60x as fast. So, if you really had an application that had performance requirements, you should be using NumPy directly and not Pandas.

### read_csv vs read_table duplication
One example of duplication is with the read_csv and read_table functions. They both do the same exact thing, read in data from a text file. The only difference is that read_csv defaults the delimiter to a comma, while read_table uses tab as its default.
    df_1=pd.read_csv('play_evaluation.csv')
    df_2=pd.read_table('play_evaluation.csv', delimiter=',')
    df_1.equals(df_2)
    **True**

The read_table function is getting deprecated and should never be used.

### isna vs isnull and notna vs notnull
The isna and isnull methods both determine whether each value in the DataFrame is missing or not. The result will always be a DataFrame (or Series) of all boolean values. These methods are exactly the same. We say that one is an alias of the other. 
The isna method was added more recently because the characters na are found in other missing value methods such as dropna and fillna. 
Confusingly, Pandas uses NaN, None, and NaT as missing value representations and not NA. notna and notnull are aliases of each other as well and simply return the opposite of isna.

    df_isna = df.isna()
    df_isnull = df.isnull()
    df_isna.equals(df_isnull)
    **True**

I only use isna and notna I use the methods that end in na to match the names of the other missing value methods dropna and fillna. You can also avoid ever using notna since Pandas provides the inversion operator, ~ to invert boolean DataFrames.


### Groupby Aggregation
There are a number of syntaxes that get used for the groupby method when performing an aggregation. I suggest choosing a single syntax so that all of your code looks the same.

Typically, when calling the groupby method, you will be performing an aggregation. This is the by far the most common scenario. When you are performing an aggregation during a groupby, there will always be three components:
- Grouping column — Unique values form independent groups
- Aggregating column — Column whose values will get aggregated; usually numeric
- Aggregating function — How the values will get aggregated (sum, min, max, mean, median, etc…)

There are a few different syntaxes that Pandas allows to perform a groupby aggregation. The following is the one I use. df.groupby('grouping column').agg({'aggregating column': 'aggregating function'})

Method 1: Here is the method I use; it handles complex cases.
    df.groupby(['Date', 'Platform']).agg({'client_id':'nunique'}).head()

![](/images/4.png)

Method 2a: The aggregating column can be selected within brackets following the call to groupby. Notice that a Series is returned here and not a DataFrame.
    df.groupby('Platform')['client_id'].agg('nunique').head()

![](/images/5.png)

Method 2b: The aggregate method is an alias for agg and can also be used. This returns the same Series as above.
    df.groupby('Platform')['client_id'].aggregate('nunique').head()

![](/images/5.png)

Method 3: You can call the aggregating method directly without calling agg. This returns the same Series as above.
    df.groupby('Platform')['client_id'].nunique().head()

![](/images/5.png)

The reason I choose this syntax is that it can handle more complex grouping problems. For instance, if we wanted to find the max and min of the math and verbal sat scores along with the average undergrad population per state we would do the following.
   df.groupby('Platform').agg({'client_id': 'nunique',
                              'experience_points': ['min', 'max'],
                              'outcome': 'count'}).round(0).head(10)


![](/images/6.png)
### Handling a MultiIndex
A MultiIndex or multi-level index is a cumbersome addition to a Pandas DataFrame that occasionally makes data easier to view, but often makes it more difficult to manipulate. 
You usually encounter a MultiIndex after a call to groupby when using multiple grouping columns or multiple aggregating columns.
   agg_dict ={'client_id': 'nunique',
            'experience_points': ['min', 'max'],
            'outcome': 'count'}
   df = df.groupby(['Date', 'Platform']).agg(agg_dict)
   df.head(10).round(0)

![](/images/7.png)
### MultiIndex in both the index and columns
Both the rows and columns have a MultiIndex with two levels; selection and further processing is difficult with a MultiIndex. There is little extra functionality that a MultiIndex adds to your DataFrame. 
They have different syntax for making subset selections and are more difficult to use with other methods. 
If you are an expert Pandas user, you can get some performance gains when making subset selections, though I typically do not like the added complexity that they come with.
We can convert this DataFrame so that only single-level indexes remain. There is no direct way to rename columns of a DataFrame during a groupby (yes, something so simple is impossible with pandas), so we must overwrite them manually.

   df.columns = ['nunique client_id', 'min experience_points',         
                   'max experience_points', 'count outcome']
   df.head()

![](/images/8.png)

From here, we can use the reset_index method to make each index level an actual column.
   df.reset_index().head()

![](/images/9.png)

Avoid using a MultiIndex. Flatten it after a call to groupby by renaming columns and resetting the index.

### similarity between groupby, pivot_table, and crosstab
Some users might be surprised to find that agroupby (when aggregating), pivot_table, and pd.crosstab are essentially identical. However, there are specific use cases for each, so all still meet the threshold for being included in a minimally sufficient subset of Pandas.

### groupby aggregation vs. pivot_table
Performing an aggregation with groupby is essentially equivalent to using the pivot_table method. Both methods return the exact same data, but in a different shape.

   sales = pd.read_csv('sales_evaluation.csv', sep=';')
   sales.head()

![](/images/10.png)

   sales.groupby(['client_id', 'store_item_name']).agg({'dollar_spent':'sum'}).head()

![](/images/11.png)

We can duplicate this data by using a pivot_table.

   sales.pivot_table(index='client_id', columns='store_item_name', 
                    values='dollar_spent', aggfunc='sum').head()

![](/images/12.png)
Notice that the values are exactly the same. The only difference is that the store item column has been pivoted so its unique values are now the column names. The same three components of a groupby are found in a pivot_table.
The grouping column(s) are passed to the index and columns parameters. The aggregating column is passed to the values parameter and the aggregating function is passed to the aggfunc parameter. 
It’s actually possible to get an exact duplication of both the data and the shape by passing both grouping columns as a list to the index parameter.

   sales.pivot_table(index=['client_id','store_item_name' ], 
                    values='dollar_spent', aggfunc='sum').head()

![](/images/13.png)

Typically, pivot_table is used with two grouping columns, one as the index and the other as the columns. But, it can be used for a single grouping column. The following produces an exact duplication of a single grouping column with groupby.

   df1 = sales.groupby('client_id').agg({'dollar_spent':'sum'}).head()
   df2 = sales.pivot_table(index='client_id', values='dollar_spent', 
                          aggfunc='sum').head()
   df1.equals(df2)
   **True**