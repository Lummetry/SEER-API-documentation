# LENS Product Explorer API documentation #

LENS Product Explorer is a multi-purpose system that targets retail and distribution horizontals. It analyzes the transactions or more generally, the events) only using the obfuscated SKU information and the tranzaction ID. Finally, the system discovers the items knowledge graph (where each item has a unique position) and creates multiple recommender systems.

## API Calls ##

Each API call has 2 required parameters:
| Key | Type | Value description |
| :--- | :--- | :--- |
| `TASK` | `string` | The task definition |
| `MAP_ID` | `string` | A string that identifies the map |

Besides these required parameters, each API call might have additional parameters.

### 1. get_items_map ###

The first API call is aimed to respond with the 2D projection of this items knowledge graph ("items map") and to __automatically__ find out the interests for which are bought the items by the end client.

#### Additional parameters: ####

| Key | Type | Value description |
| :--- | :--- | :--- |
| `PNG` | `boolean` | Specifies whether the API sends or not a PNG snapshot of the items map |
| `PNG_SIZE` | `integer` | The size (in pixels) of the PNG snapshot |

#### Request: ####

```python
{
  "TASK" : "get_items_map",
  "MAP_ID" : "20200314_162354",
  "PNG" : True,
  "PNG_SIZE" : 5000
}
```

#### Response: ####

```python
{
  "DATAFRAME_MAP" : {
    "ENCODING" : "JSON",
    "STR" : "<JSON encoded dataframe map>",
    "DESCRIPTION" : "DataFrame decoding (python code snippet): `df_items = pd.DataFrame(json.loads(str_))`, where `str_` is the value found in the json object"
  },
  
  "PNG_MAP" : {
    "ENCODING" : "base64",
    "STR" : "<base64 encoded png file>",
    "DESCRIPTION" : "PNG decoding (python code snippet): `with open(path_to_img.png, 'wb') as handle: handle.write(base64.b64decode(str_))`, where `str_` is the value found in the json object."
  },
  
  "INTERESTS_CENTERS" : [[23.73, 14.21, 0], [14.56, 45.31, 1], ...]
}
```
* `INTERESTS_CENTERS` is a list with `N` elements, where `N = nr discovered interests`. Each element in the list is also a list that contains 3 details (`x` and `y` coordinates on the 2D map and the `InterestId`)  

* Python code snippets are provided in response for decoding both the `DATAFRAME_MAP` and `PNG_MAP`.  

* The header of the dataframe map is:

| Column name | Description |
| :--- | :--- |
| `ItemId` | The obfuscated SKU code of the item |
| `IDE` | The internal LENS PE ID of the item |
| `IterestId` | The ID of the automatic discovered interest category of the item |
| `X` | The x-coordinate of the item on the 2D map |
| `Y` | The y-coordinate of the item on the 2D map |




### 2. get_interests_overview_by_score ###

Returns an overview of the automatic identified interest categories.

#### Request: ####

```python
{
  "TASK" : "get_interest_categories_overview_by_score",
  "MAP_ID" : "20200314_162354"
}
```

#### Response: ####

```python
{
  "INTEREST" : [0, 1, 2, 3, ...],
  "NR_ITEMS" : [432, 25, 14, ...],
  "PURCHASE_INDICATOR" : [54.21, 100, 1.45, ...]
}
```

The API returns a response with 3 keys. Each key has as value a list. All lists in the response have the same length. In the above example, `INTEREST` number 3 has 14 items and a `PURCHASE_INDICATOR = 1.45`. 

### 3. get_category_interests ###

Returns all the interests identified in a category defined by the category management department. In this request, a threshold (`MIN_ITEMS_PER_INTEREST`) could be set in order to return only the interests that has at least `MIN_ITEMS_PER_INTEREST` items.

#### Additional parameters: ####

| Key | Type | Value description |
| :--- | :--- | :--- |
| `CATEGORY_ID` | `integer` | A category defined by the category management department |
| `MIN_ITEMS_PER_INTEREST` | `integer` | Threshold |

#### Request: ####

```python
{
  "TASK" : "get_category_interests",
  "MAP_ID" : "20200314_162354",
  "CATEGORY_ID" : 7,
  "MIN_ITEMS_PER_INTEREST" : 5
}
```

#### Response: ####

```python
{
  "CATEGORY_INTERESTS" : [0, 323, 400]
}
```


### 4. get_interest_items ###

Returns all the items identified in an interest category.

#### Additional parameters: ####

| Key | Type | Value description |
| :--- | :--- | :--- |
| `INTEREST_ID` | `integer` | The ID of the interest category |

#### Request: ####

```python
{
  "TASK" : "get_interest_items",
  "MAP_ID" : "20200314_162354",
  "INTEREST_ID" : 323
}
```

#### Response: ####

```python
{
  "INTEREST_ITEMS" : [456223, 45773, 9912444, 18432, ...]
}
```

The reponse contains a list all obfuscated SKUs that are identified in the interest category specified in the request.


### 5. get_category_outliers ###

Returns all the outliers of a category. Specifically, the outliers are all the items that are found in interest categories that do not represent the requested cateogory.

#### Additional parameters: ####

| Key | Type | Value description |
| :--- | :--- | :--- |
| `CATEGORY_ID` | `integer` | A category defined by the category management department |

#### Request: ####

```python
{
  "TASK" : "get_category_outliers",
  "MAP_ID" : "20200314_162354",
  "CATEGORY_ID" : 7
}
```

#### Response: ####

```python
{
  "CATEGORY_OUTLIERS" : [76765, 232123, 1129011, 9122112, ...]
}
```

The reponse contains a list all obfuscated SKUs that are identified as category outliers.


### 6. get_baseline_recom ###

Returns recommendations using a statistical (baseline) engine.

#### Additional parameters: ####

| Key | Type | Value description |
| :--- | :--- | :--- |
| `ITEMS_LIST` | `integer` or `list[integer]` | Obfuscated SKUs used for starting baseline recommendations |
| `NR_ITEMS` | `integer` | The number of items recommended for each item in `ITEMS_LIST` |

#### Request: ####

```python
{
  "TASK" : "get_baseline_recom",
  "MAP_ID" : "20200314_162354",
  "ITEMS_LIST" : [232123, 9122112],
  "NR_ITEMS" : 5
}
```

#### Response: ####
```python
{
  "ITEMS" : [[898989, 928212, 7428121, 943113, 2133442], [2819122, 9343211, 2044331, 395343, 131431]]
  "SCORES": [[74, 67, 43, 21, 9], [3242, 2844, 1001, 198, 42]
}
```
