# A. Products maps API #

Single endpoint for `POST` requests: `http://<ip_address_to_be_provided>/analyze`

* ## A.1\. Get products map table
  **Request:**
  ```python
  {
    "TASK" : "get_items_map",
    "CONTEXT_ID" : <str: mandatory>, # the id of the map / context
  }
  ```
  
  **Response if the map is available:**
  ```python
  {
    "SUCCESS" : true,
    "DATAFRAME_MAP" : <str>, #the table that provides ML-generated information about the products
    "INTERESTS_CENTERS" : <list[float, float, int]> # [[x1,y1,cluster1], [x2,y2,cluster2], ...]
  }
  ```
  You can download an example response from: https://www.dropbox.com/s/n51sxn2gi0x6fog/response_example.json?dl=0
  
  **Response if the map is not generated yet:**
  ```python
  {
    "SUCCESS" : false,
    "REASON" : "Try again later. The map is not generated yet (maybe there is a trraining in progress)"
  }
  ```
  
  **Understanding the response:**
    * ### `“DATAFRAME_MAP"`
      The string returned for `"DATAFRAME_MAP"` represents the json codification of a table with x columns and y rows. All needed to recreate the table is to generate the dictionary,  `dct = json.loads(response["DATAFRAME_MAP"])` and then use a library that handles dataframes (in python the best way to do it is using `pandas`: `df = pandas.DataFrame(dct))`. You can download a csv file that is generated based on `response["DATAFRAME_MAP"]` from: https://www.dropbox.com/s/jz9euj2lw6oi3tt/decoded_dataframe_example.csv?dl=0
      
      _The head of the table looks as follows:_
      ```python
          ItemId  IDE   Freq  InterestId      X      Y
      0   545535    0  26079          -1  -3.14  11.37
      1   398648    1  20894           0 -39.36   8.45
      2   563633    2  15816           1   9.70  21.34
      3   576732    3  14902          -1  37.50  24.44
      4   406083    4  12334          -1 -16.86   8.51
      5   602874    5  10537          -1  37.44  24.51
      6   589207    6   9816          -1  31.49  13.40
      7   258901    7   9507           2 -59.15 -20.47
      8   486656    8   9500          -1  37.44  32.09
      9   527499    9   9138           9  33.00  15.15
      10  533559   10   9117          -1  -7.31 -43.69
      11  524657   11   8949          -1  25.81  27.62
      12  599979   12   8623           3  35.00  46.06
      13  651356   13   8516          -1  13.96  25.45
      14  466968   14   8056          -1  36.96  26.99
      15  213201   15   7944           4  37.13  30.41
      16  616607   16   7915          -1  39.56  20.04
      17  259150   17   7634          -1 -56.05   5.88
      18  589279   18   7560          77  36.03  31.60
      19  538942   19   7553           5  35.04  15.56
      ```
      
      _The tail of the table looks as follows:_
      ```python
              ItemId     IDE  Freq  InterestId   X   Y
      142937  732521  142937     1          -1 NaN NaN
      142938  803789  142938     1          -1 NaN NaN
      142939  801273  142939     1          -1 NaN NaN
      142940  785036  142940     1          -1 NaN NaN
      142941  690413  142941     1          -1 NaN NaN
      142942  375936  142942     1          -1 NaN NaN
      142943  780657  142943     1          -1 NaN NaN
      142944  803929  142944     1          -1 NaN NaN
      142945  801901  142945     1          -1 NaN NaN
      142946  786783  142946     1          -1 NaN NaN
      ```
      
      _Properties of the table:_ 
      1. the number of rows is equal to the number of the unique items/products in the provided training dataset
      2. The column `ItemId` contains the items/products skus provided in the training dataset
      3. The column `IDE` is an internal continous id in range `[0, nr_products)` given based on the frequency (the higher the frequency `Freq`, the lower the `IDE`)
      4. The column `Freq` is the frequency of the item in transactions (an item with frequency 1 was ever bought just once)
      5. The column `InterestId` is the id of the need/interest/category/cluster in which the model places the item. Based on this field the points are colored on the map. If `InterestId` is -1, it means that the model did not handled to place the item in a certain category (whether because there are very few transactions involving that item, or because the pattern cannot be identified well).
      6. The columns `X` and `Y` are the items coordinates on the 2D map. The NaN values are for the items that cannot be displayed on the map (always these are the ones with the lowest frequencies). Even they cannot be displayed on the map, they are taken into consideration when we compute similarity, complementarity or other metrics

   * ### `“INTERESTS_CENTERS"`

     For each generated interest/cluster, the model will compute also the center in terms of x and y coordinates. Therefore, the value for `“INTERESTS_CENTERS“` looks as follows:
     
     ```python
     [
       [1.2455, 2.83844, 0],
       [2.6724, 1.24341, 1],
       ...
       [31.8912, 67.12134, 1788]
     ]
     ```
     
     The single purpose of this field is when the center of each cluster should be drawn on the map. Otherwise, it can be discarded.
     
     _Properties of the list:_ 
     1. The number of elements in the list is equal to the number of the generated clusters (i.e. `len(response["INTERESTS_CENTERS"]) == df['InterestId'].max()+1`)
     2. Each element in the list is a list with 3 objects: x, y, and id of the cluster.
     3. The ids of the clusters (found on the 3rd object of each list element) are identical to the ones found in the table on the `"InterestId"` column

    
   _Info:_ 
   ```python
   From our experience, the easiest way to integrate the map in a website is using bokeh: https://docs.bokeh.org/en/latest/docs/user_guide/bokehjs.html
   ```
     
   _Code snippet for rendering a map in python:_
   ```python
   from bokeh.plotting import figure
   from bokeh.models import ColumnDataSource, HoverTool,\
     BoxZoomTool, ResetTool, WheelZoomTool, PanTool,\
     Arrow, VeeHead, CustomJS, Label, LassoSelectTool, Slider
   from bokeh.models.widgets import TableColumn, DataTable, TextEditor,\
     Button, MultiSelect

   HEIGHT = ...
   WIDTH = ...

   source_data =  dict(x=[], y=[], color=[], name=[], item_id=[], interest_id=[])
   source_data["x"] = df["X"].values
   source_data["y"] = df["Y"].values
   source_data["color"] = compute_colors(df["InterestId"].values)
   source_data["name"] = get_names(df["ItemId"].values)
   source_data["item_id"] = df["ItemId"].values
   source_data["interest_id"] = df["InterestId"].values

   tooltips = [("Nume", "@name"), ("ItemId", "@item_id"), ("Category", "@interest_id")]
   source = ColumnDataSource(data=source_data)
   p = figure(
     plot_height=HEIGHT,
     plot_width=WIDTH,
     title="",
     tools=[BoxZoomTool(), WheelZoomTool(), ResetTool(), PanTool(), LassoSelectTool()]
   )
   p_scatter = p.scatter(
     x="x", y="y", source=source,
     radius = 0.25,
     color="color", line_color = None
   )
   hover = HoverTool(tooltips=tooltips, renderers=[p_scatter])
   p.add_tools(hover)
   ...
   ```
     
* ## A.2\. Adiacent calculus based on the map
  * ### A.2.1\. Get all the items in a certain cluster
    Even though this calculus can be done based on the provided table, we also provide this functionality via API. 
   
    TODO
     

# B. Training new maps #
  For training new maps, training data is needed (transactions). We will want to train new maps when new filters are generated (e.g. the map of the products for the Valentine’s day season, the map of the products for location x, etc.). The filtered transactions should be dumped in a database view in order to fetch the data using a simple query `"SELECT * FROM <view>"`.
 
After the filtered data is dumped in the view, you can make an API request (`POST http://<ip_address_to_be_provided>/product_explorer`)

  **Request:**
  ```python
  {
    "TASK" : "train_map",
    "CONTEXT_ID" : <str: mandatory>, # the id of the map / context
    "TABLE_INP" : <str: mandatory>, # the name of the table / view in the database where the filtered transactional data is stored
    "TABLE_OUT" : <str: mandatory>, # the name of the table / view in the database where the map will be saved after training
    "HOUR" : <int: optional [default 19]>, # the hour when this particular training will be scheduled 
    "MINUTE" : <int: optional [default 0]> # the minute when this particular training will be scheduled
  }
  ```
  
  Observation: The fields `"HOUR"` and `"MINUTE"` are used to schedule the training. By default any training is scheduled to start in the same day when the request was sent at 16:00. Attention! The time of the machines were the training is performed is usually GMT.
  
  **Response:**
  ```python
  {
    "message" : "success"
  }
  ```
 
  
  
