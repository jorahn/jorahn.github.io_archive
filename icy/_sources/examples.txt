Examples
========

Parsing many HTML-Files
-----------------------
Download and unzip the `Baby Names dataset
<https://developers.google.com/edu/python/google-python-exercises.zip>`_ (part of the great 
`Google Python Course
<https://developers.google.com/edu/python/exercises/baby-names>`_). From the archive we'll just use the `/babynames`-folder (you can leave the .py-files and the subfolder in place).

The dataset consists of 10 regular html-files taken from http://www.ssa.gov/. If you preview the files, you'll find tables embedded in the html with three columns `Rank`, `Male name`, `Female name` and 1000 rows per year.

::

  >>> import icy
  >>> icy.preview('~/Downloads/google-python-exercises/babynames/')

This results in a flood of data on your screen. Obviously the parser detects more tables than just the names. If you scroll through the preview result you'll find results like this:

::

  File: baby2008.html_baby2008.html_2
  
  <class 'pandas.core.frame.DataFrame'>
  Int64Index: 1002 entries, 0 to 1001
  Data columns (total 3 columns):
  0    1002 non-null object
  1    1001 non-null object
  2    1001 non-null object
  dtypes: object(3)
  memory usage: 31.3+ KB
  
  COLUMN               | first 5 VALUES
  ----------------------------------------
  0                    | ['Rank', '1', '2', '3', '4']
  1                    | ['Male name', 'Jacob', 'Michael', 'Ethan', 'Joshua']
  2                    | ['Female name', 'Emma', 'Isabella', 'Emily', 'Madison']

This looks roughly like what we are aiming for. By passing the parsing arguments **header = 0** and **index_col = 0** takes care of identifying the correct column names and using the `Rank` column as the index.

Since the html-files contain more <table> tags, we have a number of results we would want to ignore. Luckily you can identify the interesting tables by filtering for tables featuring `summary="Popularity for top 1000"`. There keyword to accomplish this is **attrs = {'summary': 'Popularity for top 1000'}**.

To get rid of the error, that the file `babynames.py` could not be parsed we can pass **filters = '.html'**.

::

  >>> import icy
  >>> src = '~/Downloads/google-python-exercises/babynames/'
  >>> icy.preview(src,
  cfg = {
    'filters': '.html',
    'default': {
      'header': 0,
      'index_col': 0,
      'attrs': {'summary': 'Popularity for top 1000'}
  }})

This results in a preview of the first rows of all 10 tables. To simplify generating and testing the required parsing arguments, you can provide the location of a YAML-file to the `cfg`-keyword. A `babynames_read.yml` for this example would be:

::

  filters:
    - '.html'
  default:
    attrs: {summary: Popularity for top 1000}
    header: 0
    index_col: 0

Now run the whole thing:

::

  >>> import icy
  >>> src = '~/Downloads/google-python-exercises/babynames/'
  >>> cfg = '~/Downloads/google-python-exercises/babynames_read.yml'
  >>> data = icy.read(src, cfg)

**data** is a dict of pandas.DataFrames with 10 keys and a total memory usage of 234.6 KB.

**Note:** We used the 'default'-key to apply the parsing arguments to every file. For more heterogenous data, you can specify different parsing arguments by using the filename (e.g. 'baby1990.html') as a key. If you specify 'default' and specifig arguments, the 'default' is still applied to all files but the specific arguments override the 'default'.


Parsing many compressed CSV-Files
---------------------------------
Download the `Lahman Baseball dataset
<http://seanlahman.com/files/database/lahman-csv_2015-01-24.zip>`_ (from Sean Lahman's `extensive Baseball resources
<http://www.seanlahman.com/baseball-archive/statistics/>`_) and the `Catapillar Tube Pricing dataset
<https://www.kaggle.com/c/caterpillar-tube-pricing/data>`_ (from `Kaggles Catapillar competition
<https://www.kaggle.com/c/caterpillar-tube-pricing>`_). The Lahman dataset consists of 24 csv-files and a `readme.txt` while the Caterpillar dataset features 21 csv-files.

::

  >>> import icy
  >>> icy.preview('~/Downloads/lahman-csv_2015-01-24.zip')
  # output for lahman-csv_2015-01-24.zip
  >>> icy.preview('~/Downloads/data.zip')
  # output for data.zip

Again a lot of data appears on your screen for each of the two datasets. Most of the results seem quite sensible but we can still do a little better with this `lahman_read.yml` (ignore the readme.txt and parse date-columns):

::

  filters:
    - '.csv'
  Master.csv:
    parse_dates: ['debut', 'finalGame']
  
and this `cat_read.yml` (parse custom nan- and boolean-values and date-columns):

::

  default:
    na_values: ['NA', 'NONE']
    true_values: ['Yes', 'Y']
    false_values: ['No', 'N']
  test_set.csv:
    parse_dates: ['quote_date']
  train_set.csv:
    parse_dates: ['quote_date']
  
Now run the whole thing:

::

  >>> import icy
  >>> src = '~/Downloads/lahman-csv_2015-01-24.zip'
  >>> cfg = '~/Downloads/lahman_read.yml'
  >>> data = icy.read(src, cfg)
  # data for lahman-csv_2015-01-24.zip
  >>> src = '~/Downloads/data.zip'
  >>> cfg = '~/Downloads/cat_read.yml'
  >>> data = icy.read(src, cfg)
  # data for data.zip

**data** is a dict of pandas.DataFrames with 24 keys and a total memory usage of 82.3 MB or 21 keys and a total memory usage of 11.0 MB respectively.
