# LENS Product Explorer API documentation #

LENS Product Explorer is a multi-purpose system that targets retail and distribution horizontals. It analyzes the transactions or more generally, the events) only using the obfuscated SKU information and the tranzaction ID. Finally, the system discovers the items knowledge graph (where each item has a unique position) and creates multiple recommender systems.

## 1. Recommender API ##

Each API call has 3 __required__ parameters:
| Key | Value type | Value description |
| :--- | :--- | :--- |
| `RECOM_TYPE` | `string` | The entry-point name of the recommendation type|
| `CONTEXT_ID` | `string` | A string that identifies the context (a location in a season, for example) |
| `NR_ITEMS` | `integer` | The number of items to be recommended |

Additionally, each API call may have 2 __optional__ parameters:
| Key | Value type | Value description |
| :--- | :--- | :--- |
| `FILTER_ITEMS` | `list[integer]` | Obfuscated SKUs from which the recommedations will be chosen |
| `EXCLUDE_ITEMS` | `list[integer]` | Obfuscated SKUs which will be excluded from recommedations |

Finally, for each recommendation type may appear parameters such as `START_ITEMS`, `NR_INTERESTS`, `CURRENT_BASKET`, `HISTORY` and `USER_ID` which will be detalied wherever they will appear.

Below is defined the parameters summary table. It specifies for each recommendation type, which parameters are __required (R)__, which are __optional (O)__ and which are __not applicable (N/A)__:  
| Recom | `RECOM_TYPE` | `CONTEXT_ID` | `NR_ITEMS` | `START_ITEMS` | `NR_INTERESTS` | `CURRENT_BASKET` | `HISTORY` | `USER_ID` | `FILTER_ITEMS` | `EXCLUDE_ITEMS` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| [get_same_interest_recom](#111-get_same_interest_recom) | R | R | R | R | N/A | N/A | N/A | N/A | O | O |
| [get_near_interest_recom](#112-get_near_interest_recom) | R | R | R | R | R | N/A | N/A | N/A | O | O |
| [get_complementary_interests_recom](#113-get_complementary_interests_recom) | R | R | R | R | R | N/A | N/A | N/A | O | O |
| [get_short_term_insights_recom](#121-get_short_term_insights_recom) | R | R | R | N/A | N/A | R | N/A | N/A | O | O |
| [get_short_term_last_minute_recom](#122-get_short_term_last_minute_recom) | R | R | R | N/A | N/A | R | N/A | N/A | O | O |
| [get_long_term_insights_recom](#131-get_long_term_insights_recom) | R | R | R | N/A | N/A | R | R | N/A | O | O |
| [get_long_term_last_minute_recom](#132-get_long_term_last_minute_recom) | R | R | R | N/A | N/A | R | R | N/A | O | O |
| [get_long_term_next_basket_recom](#133-get_long_term_next_basket_recom) | R | R | N/A | N/A | N/A | N/A | R | R | O | O |

## 1.1 Single-item recommendations

The recommendations in this section are computed given a single item. The item should be specified as value for key __`START_ITEMS`__. The engine can handle multiple items in the same request and therefore will be recommended `NR_ITEMS` items for each item in `START_ITEMS`. Below is found the description of the __`START_ITEMS`__ parameter:

| Key | Value type | Value description |
| :--- | :--- | :--- |
| `START_ITEMS` | `integer` or `list[integer]` | Obfuscated SKU(s) used for starting single-item recommendations |

### 1.1.1 get_same_interest_recom ###

Returns recommendations with the best matched items from the same interest category of the starting item.

#### Request: ####

```python
{
  "RECOM_TYPE" : "get_same_interest_recom",
  "CONTEXT_ID" : "20200314_162354",
  "START_ITEMS" : [512762],
  "NR_ITEMS" : 10
}
```

#### Response: ####
```python
{
  "ITEMS" : [[599477, 599484, 408464, 599482, 640880, 647459, 428259, 610448, 473932, 457259]]
}
```

### 1.1.2 get_near_interest_recom ###

Returns recommendations with the best matched items from interest categories which are in the neighborhood of the interest category of the starting item.

#### Additional parameters: ####

| Key | Value type | Value description |
| :--- | :--- | :--- |
| `NR_INTERESTS` | `integer` | The number of *neighbor* interest categories for each item in `START_ITEMS` |

#### Request: ####

```python
{
  "RECOM_TYPE" : "get_near_interest_recom",
  "CONTEXT_ID" : "20200314_162354",
  "START_ITEMS": [512762, 673224],
  "NR_ITEMS": 2,
  "NR_INTERESTS": 3
}
```

#### Response: ####
```python
{
  "ITEMS": [[[599477, 598940],
             [587387, 538515],
             [477113, 646869]],
             
            [[309196, 605327],
             [582147, 350641],
             [892344, 812242]]
            ]
}
```

### 1.1.3 get_complementary_interests_recom ###

Returns recommendations with the best matched items from interest categories which are often bougth together (on the same receipt) with the interest category of the starting item.

#### Additional parameters: ####

| Key | Value type | Value description |
| :--- | :--- | :--- |
| `NR_INTERESTS` | `integer` | The number of *complementary* interest categories for each item in `START_ITEMS` |

#### Request: ####

```python
{
  "RECOM_TYPE" : "get_near_interest_recom",
  "CONTEXT_ID" : "20200314_162354",
  "START_ITEMS": [512762, 673224],
  "NR_ITEMS": 2,
  "NR_INTERESTS": 3
}
```

#### Response: ####
```python
{
  "ITEMS": [[[982455, 184553],
             [587387, 538515],
             [929212, 355533]],
             
            [[190223, 100024],
             [582147, 350641],
             [892344, 812242]]
            ]
}
```

### 1.2 Short-term recommendations

The recommendations in this section are computed given a basket of items. The basket should be specified as value for key __`CURRENT_BASKET`__. Below is found the description of the __`CURRENT_BASKET`__ parameter:

| Key | Value type | Value description |
| :--- | :--- | :--- |
| `CURRENT_BASKET` | `list[integer]` | Obfuscated SKUs of items found in a shopping basket |

These recommendations are suitable for anonymous users or for new users (that do not have a shopping history).

### 1.2.1 get_short_term_insights_recom

Starting from the current shopping basket, the neural model returns 1 or more items which are considered to be most suited to be bought together with the current items in the shopping basket. The name of this recommendation type is "insights" because, probably, the user does not know "consciously" what might want to buy and then, the system cames to help him, giving insights.

#### Request: ####

```python
{
  "RECOM_TYPE": "get_short_term_insights_recom",
  "CONTEXT_ID": "20200314_162354",
  "CURRENT_BASKET": [442974, 442973],
  "NR_ITEMS": 3
}
```

#### Response: ####
```python
{
  "ITEMS": [402010, 409228, 424328],
  "SCORES" : [1.0, 0.995, 0.99]
}
```

### 1.2.2 get_short_term_last_minute_recom

At checkout, the system shows one or more “last-minute” items which are the most suited to be bought with the current shopping basket.

#### Request: ####

```python
{
  "RECOM_TYPE": "get_short_term_last_minute_recom",
  "CONTEXT_ID": "20200314_162354",
  "CURRENT_BASKET": [442974, 442973, 981354],
  "NR_ITEMS": 3
}
```

#### Response: ####
```python
{
  "ITEMS": [901345, 8924442, 424328],
  "SCORES" : [1.0, 0.995, 0.99]
}
```


### 1.3 Long-term recommendations

The recommendations in this section are computed given a basket of items (the current one) __and__ a history of shopping baskets. The current basket should be specified as value for key __`CURRENT_BASKET`__ and the history should be specified as value for key __`HISTORY`__. Below is found the description of these parameters:

| Key | Value type | Value description |
| :--- | :--- | :--- |
| `CURRENT_BASKET` | `list[integer]` | Obfuscated SKUs of items found in the current shopping basket |
| `HISTORY` | `list[list[integer]]` | Obfuscated SKUs of items found in past shopping baskets | 

These recommendations are suitable for users that do have a shopping history.

### 1.3.1 get_long_term_insights_recom

Same description as [get_short_term_insights_recom](#121-get_short_term_insights_recom).

#### Request: ####

```python
{
  "RECOM_TYPE": "get_long_term_insights_recom",
  "CONTEXT_ID": "20200314_162354",
  "CURRENT_BASKET": [442974, 442973],
  "HISTORY" : [[894556], [298244, 345854], [29445, 2835445, 9355422], [385534]],
  "NR_ITEMS": 3
}
```

#### Response: ####
```python
{
  "ITEMS": [8935335, 8392233, 4548222],
  "SCORES" : [0.98, 0.95, 0.945]
}
```

### 1.3.2 get_long_term_last_minute_recom

Same description as [get_short_term_last_minute_recom](#122-get_short_term_last_minute_recom).

#### Request: ####

```python
{
  "RECOM_TYPE": "get_long_term_insights_recom",
  "CONTEXT_ID": "20200314_162354",
  "CURRENT_BASKET": [442974, 442973, 981354],
  "HISTORY" : [[894556], [298244, 345854], [29445, 2835445, 9355422], [385534]],
  "NR_ITEMS": 3
}
```

#### Response: ####
```python
{
  "ITEMS": [901345, 8924442, 424328],
  "SCORES" : [0.97, 0.932, 0.90]
}
```


### 1.3.3 get_long_term_next_basket_recom

Additional to [get_long_term_insights_recom](#131-get_long_term_insights_recom) and [get_long_term_last_minute_recom](#132-get_long_term_last_minute_recom), that do not take into account the user identity, this recommendations type is user-specific. More detalied, this is a functionality that predicts the next basket of a specific user and therefore, the prediction can be used as content for email newsletter or as content for "welcome page" when the user logs in again in the platform.

Below is found the description of the __`USER_ID`__ parameter:

| Key | Value type | Value description |
| :--- | :--- | :--- |
| `USER_ID` | `integer` | Obfuscated user ID for which the recommendations are computed |


### Request: ###

```python
{
  "RECOM_TYPE": "get_long_term_insights_recom",
  "CONTEXT_ID": "20200314_162354",
  "USER_ID": 6712,
  "HISTORY" : [[894556], [298244, 345854], [29445, 2835445, 9355422], [385534]]
}
```

### Response: ###
```python
{
  "ITEMS": [909344, 823894, 8843434, 2737544],
  "SCORES" : [0.97, 0.932, 0.90]
}
```

The number of response `ITEMS` can be variable from a prediction to another, because the model predicts the next basket.


## 2. Category Interests API ##

Each API call has 2 required parameters:
| Key | Value type | Value description |
| :--- | :--- | :--- |
| `TASK` | `string` | The task definition |
| `MAP_ID` | `string` | A string that identifies the map |

Besides these required parameters, each API call might have additional parameters.

### 2.1 get_items_map ###

The first API call is aimed to respond with the 2D projection of this items knowledge graph ("items map") and to __automatically__ find out the interests for which are bought the items by the end client.

#### Additional parameters: ####

| Key | Value type | Value description |
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




### 2.2 get_interests_overview_by_score ###

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

### 2.3 get_category_interests ###

Returns all the interests identified in a category defined by the category management department. In this request, a threshold (`MIN_ITEMS_PER_INTEREST`) could be set in order to return only the interests that has at least `MIN_ITEMS_PER_INTEREST` items.

#### Additional parameters: ####

| Key | Value type | Value description |
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


### 2.4 get_interest_items ###

Returns all the items identified in an interest category.

#### Additional parameters: ####

| Key | Value type | Value description |
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


### 2.5 get_category_outliers ###

Returns all the outliers of a category. Specifically, the outliers are all the items that are found in interest categories that do not represent the requested cateogory.

#### Additional parameters: ####

| Key | Value type | Value description |
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

## 3. Business Intelligence tool ##

#### Request ####

``` python
{
      'groupby':['per_need', ‘per_month’],
      'filters':[
        {
          'start_year':'2015',
          'start_date':{
              'month':'4',
              'day':'10'
           },
           'no_days':'10'
        },
        {
              'category':['1'] 
        },
      ]
      'plot':0,
}
```

#### Response ####
``` python
[
  {'data':[
    {'TimeStamp': '2019-04', 'needs_id': '1', 'percent': 0.19}, 
    {'TimeStamp': '2018-04', 'needs_id': '1', 'percent': 0.15},
    {'TimeStamp': '2020-04', 'needs_id': '1', 'percent': 0.12},
    {'TimeStamp': '2017-04', 'needs_id': '1', 'percent': 0.09},
    {'TimeStamp': '2016-04', 'needs_id': '1', 'percent': 0.05},
    {'TimeStamp': '2019-04', 'needs_id': '0', 'percent': 0.05}, 
    {'TimeStamp': '2018-04', 'needs_id': '0', 'percent': 0.04}, 
    {'TimeStamp': '2019-04', 'needs_id': '2', 'percent': 0.03}, 
    {'other-116': 0.28}],
   'plot': ''}, 
  {'data': [
    {'TimeStamp': '2020-03', 'needs_id': '1', 'percent': 0.05}, 
    {'TimeStamp': '2019-12', 'needs_id': '1', 'percent': 0.05},
    {'TimeStamp': '2019-11', 'needs_id': '1', 'percent': 0.05}, 
    {'TimeStamp': '2020-01', 'needs_id': '1', 'percent': 0.04}, 
    {'TimeStamp': '2018-12', 'needs_id': '1', 'percent': 0.04}, 
    {'TimeStamp': '2020-02', 'needs_id': '1', 'percent': 0.03}, 
    {'TimeStamp': '2018-11', 'needs_id': '1', 'percent': 0.03}, 
    {'TimeStamp': '2020-04', 'needs_id': '1', 'percent': 0.03}, 
    {'other-44': 0.6799999999999999}], '
  plot': ''}
]
```
#### Request Parameters ####

##### groupby field: list  #####

| Field    | Field_description    |
| :--- | :--- | 
| per_day | Applies group_by timestamp with a frequency of one day |
| per_hour | Applies group_by timestamp with a frequency of one hour |
| per_month | Applies group by timestamp with a frequency of one month |
| per_site | Applies group by site_id |
| per_need | Applies group by need_id |
| per_item | Applies group by item_id |
| per_category | Applies group by category_id |

##### filter field: list(dict)  #####
    The filter is applied using 'and' logic
 | Field    | Value_type    | Field_description    | 
 | :---------- | :---------- | :--- |
 | site | list(int) [site_id1, site_id2 ...] | Only the data from the mentioned site ids list is used |
 | start_year | int | Only the data later than the start year is used |
 | end_year | int | Only the data earlier than the start year is used |
 | start_month | int{0:11} | Only the data with the month > start_month is used |
 | end_month | int{0:11} | Only the data with the month < start_month is used |
 | item | list(int) [item_id1, item_id2, …] | Only the data from the mentioned item ids list is used |
 | need | list(int) [need_id1, need_id2 ...] | Only the data from the mentioned need ids list is used |
 | category | list(int) [category_id1, category_id2 …] | Only the data from the mentioned category ids list is used |
 | Start_date :{  'month',  'day'  }| dict(  'month': int{0:11},  'day':int  ) |Specifies the start date of the season. *Only applicable if start_date and no_days are used together. Used for seasonal filtering* |
 |  No_days | int | Specifies the number of days the season lasts. *Only applicable if start_date and no_days are used together. Used for seasonal filtering* |
   
 



