# A. Products maps API #

Single endpoint for `POST` requests: `http://<ip_address_to_be_provided>/product_explorer`

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
    
    * `“DATAFRAME_MAP“`
      The string returned for `"DATAFRAME_MAP"` represents the json codification of a table with x columns and y rows. All needed to recreate the table is to generate the dictionary,  `dct = json.loads(response["DATAFRAME_MAP"])` and then use a library that handles dataframes (in python the best way to do it is using `pandas`: `df = pandas.DataFrame(dct))`. You can download a csv file that is generated based on `response["DATAFRAME_MAP"]` from: https://www.dropbox.com/s/jz9euj2lw6oi3tt/decoded_dataframe_example.csv?dl=0
