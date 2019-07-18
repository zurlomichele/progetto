# Object Oriented Programming Project
The main purpose of our project was the creation of a JAVA application capable of modeling a specific assigned data-set into objects (defining a set of classes) and then filter and get statistics from the data.
The data-set assigned by the lecturers shows the funding, divided by nation, provided by the European Union.
To meet these purposes, our application is based on the following features:
- **on startup:** downloads the data-set (which contains data in CSV format starting from the address provided after decoding the JSON which contains the URL useful for downloading the file). This operation is carried out only at the start of the application, moreover, where the dataset is already present in memory, it is not downloaded again;
- **at the end of the download:** performs data parsing by mapping each data-set record into an object of a class, paying particular attention to the different separator characters;
The CSVParser class is used to read and analyze the CSV file and to fill a payment vector and a metadata vector.
Each row represents a loan and each column header represents a metadata.
Each line is described by the following attributes:
1)  Country 
2)  NUTS1_ID
3)  NUTS2_ID
4)  NUTS2_name
5)  Fund
6)  Year 
7)  Programming_Period
8)  EU_Payment_annual
9)  Modelled_annual_expenditure
10) Standard_Deviation_of_annual_expenditure
11) Standard_Error_of_modelled_annual_expenditure


We decided to represent the column "Programming Period" of the CSV as a separated object in such a way that
a row of the CSV is representing by an object of the class "Payment" and each of these contain an object of the class "ProgrammingPeriod".
This choice gave to the user the possibility to filter the data-set also for the start/end of the Programming Period stated in the CSV.

- **on request:**  using API REST GET or POST (with separate routes) using Spring Boot are returned:  
  - metadata (JSON format) or list of attributes and type;  
  - data (JSON format);  
  - statistics on the data (JSON format) that specifying the attribute on which to perform the computation (data column) such as:
   1) Numbers: avg, min, max, dev std, sum, count  
   2) Strings: Counting of unique elements (for each unique element the number of occurrences is indicated);  
Finally, the restitution foresees the possibility to specify during the request a series of filters on attributes with conditional and logical operators.
Next, the various requests that can be performed with relevant examples will be listed.
Examples refer to the query or body of POST (JSON).
Furthermore, the implementation methods of the statistics and filters are shown and some examples of tests useful for verifying the implemented functions are listed.

WARNING: there are some wrong fields and therefore some data is managed in a dedicated way in order to be corrected. Then follow these manual manipulation operations on the CSV data:
```
public static String convString(String s) {
         if (s.equals(""))
             s = "0";
         return s;
```

Furthermore, regular expressions are adopted in order to obtain a correct reading of the file. Following is the syntax of the RegEx pattern used:

```
private String cvsSplitBy = ",(?=(?:[^\"]*\"[^\"]*\")*[^\"]*$)";
```

## GET REQUESTS

### /showdataset
This request return all the data-set.

### /showmetadata
This request return all header of the data-set.

### /statistics/{fieldName}
Gives statistics on numbers fields of our CSV, based on the class  _DataStatistics_:

-   Average
-   Minimum
-   Maximum
-   Standard Deviation
-   Sum

NOTE: this command must be used only on numeric fields!

```
results = calculate(store);
```

```
 private Vector<Object> calculate(Vector<Integer> store) {
        Integer count = store.size();
        Integer min = store.get(0);
        Integer max = store.get(0);
        Double avg = (double) 0;
        Long sum = (long) 0;
        Double std = (double) 0;
        Vector<Object> results = new Vector<Object>();
        for (Integer item : store) {
            avg += item;
            if (item < min)
                min = item;
            if (item > max)
                max = item;
            sum += item;
        }
        avg = avg / count;

        for(Integer item:store){
            std+=(item-avg)*(item-avg);
        }
        std = Math.sqrt(std/(count));

        results.add(avg);
        results.add(min);
        results.add(max);
        results.add(std);
        results.add(sum);
        return results;
    }   
```

```
return new DataStatistics((double)results.get(0), (int)results.get(1), (int)results.get(2), (double)results.get(3), (long)results.get(4));
```

_example of use:_

> localhost:8080/statistics/Year

_response:_

```
{
        "avg": 2004.8884111095447,
        "min": 1986,
        "max": 2016,
        "std": 6.545637133705266,
        "sum": 56882694
}

```

### count/{fieldName}

Returns the number of times the string of the specified field occourred.
It needs a field name in the query path and a value to confront in the query params.
```
(@PathVariable String fieldName, @RequestParam(value = "value") String value)
```
_example of use:_

> localhost:8080/count/NUTS2_name?value=Marche

_response:_

```
{
    "count": 136
}
```

## POST REQUESTS

### /filter
This is a general filter command using a POST request.
In the body of the JSON request of the user, you need to specify a field where to apply the filter, a conditional operator and an input value. Sending the request gets you the filtered data-set, showing all the occurrences that respect given rules.
If it is found a logical operator ("$or" or "$and") multiple filters, represented by an array of JSON request, are applied at the data-set on the specified fields.
The conditional operator "$or" filter the date-set for each JSON request merging all the occurrences given rules.
The conditional operator "$and" filter the data-set recursively for each JSON request, applying filter rules on the previous filtered data-set until all requests are satisfied.

_$or code:_

```
for (Object obj : jsonArray) {
                filterParam.readFields(obj);
                appOr = paymentService.filter(paymentService.getPayments(), filterParam);
                for (Payment item : appOr) {
                    if (!app.contains(item))
                        app.add(item);
                }
            }
```    
_$and code:_

```
 for (Object obj : jsonArray) {
                filterParam.readFields(obj);
                // iteration on filter method, for each new cycle we use as vector to
                //filter the output vector from the precedent cycle
                app = paymentService.filter(app, filterParam);
            }
        }
```               
_example of use:_

> localhost:8080/filter

_JSON body:_
```  
{
            "fieldName": "Fund",
            "operator": "==",
            "value": "ERDF"
 }
 ```  
 
_JSON body:_
```  
 {
     "$and": [
         {
             "fieldName": "PeriodStart",
             "operator": ">=",
             "value": "2000"
         },
         {
             "fieldName": "PeriodEnd",
             "operator": "<=",
             "value": "2006"
         }
     ]
 }
``` 
_JSON body:_
```  
  {
      "$or": [
          {
              "fieldName": "EU_Payment_annual",
              "operator": ">",
              "value": "20000"
          },
          {
              "fieldName": "EU_Payment_annual",
              "operator": "<",
              "value": "90000"
          }
      ]
  }
```  
This multiple filter example also implements the conditional operator between, moreover is possible to apply this filter with different field names.

_JSON body:_
```  
{
    "$or": [
        {
            "fieldName": "EU_Payment_annual",
            "operator": ">",
            "value": "2000000"
        },
        {
            "fieldName": "Modelled_annual_expenditure",
            "operator": "<",
            "value": "90000"
        }
    ]
}
 ```  
Another example of multiple filter using more JSON objects is:
 ```  
{
    "$and": [
        {
            "fieldName": "PeriodStart",
            "operator": ">",
            "value": "2005"
        },
        {
            "fieldName": "Country",
            "operator": "==",
            "value": "IT"
        },
        {
            "fieldName": "Standard_Deviation_of_annual_expenditure",
            "operator": "<=",
            "value": "100000"
        },
    ]
}
 ```  
### /filter/statistics/{fieldName}
This filter gives statistics on a certain field name as the one explained before, but this is using a POST request and thanks to that is possible to get statistics on a filtered list of payments.
In the body of the JSON request is possible to specify filter rules with the same syntax used for "/filter" command.

_example of use:_

> localhost:8080/filter/statistics/PeriodEnd

_JSON body:_
```
{
    "$and": [
     
        {
            "fieldName": "NUTS2_name",
            "operator": "==",
            "value": "Puglia"
        },
        {
            "fieldName": "Standard_Error_of_modelled_annual_expenditure",
            "operator": "<=",
            "value": "100000"
        }
    ]
}
```
_response:_

```
{
    "avg": 2000.9354838709678,
    "min": 1993,
    "max": 2013,
    "std": 7.079891943183822,
    "sum": 124058
}
```

### Error handler
Our application is able to provide the user some information about errors that might occur caused by invalid input or misspelled words in the various requests.
Here's some example of error notification:
- > localhost:8080/statistics/Foobar

    The application return a BAD REQUEST with the following message:
       
     ``` "Il metodo-Foobar-non esiste" ```  
- > localhost:8080/statistics/Country

    The application return a BAD REQUEST because it isn't possible to get statistics from a text field. The message displayed is:
       
    ``` "Operazione di casting non consentita" ```
      
- > localhost:8080/filter/

    _JSON body:_

   ```
    {
        "fieldName": "Fond",
         "operator": "==",
         "value": "ERDF"
    }
   ```
   The application return a BAD REQUEST with the following message:
   
   ```"Il metodo-Fond-non può essere trovato" ```
- > localhost:8080/filter/

    _JSON body:_
    ```
    {
    "fieldName": "Fund",
    "operator": ">=",
    "value": "ERDF"
    }            
    ```
    The application return a BAD REQUEST because it isn't possible to apply the specified
     operator to a string field. It displays the following message:
                                   
   ``` "Operatore non conforme, può essere solo ==" ```
   
- > localhost:8080/filter/

    _JSON body:_
    ```
    {
    "or": [
     
        {
            "fieldName": "NUTS2_name",
            "operator": "==",
            "value": "Puglia"
        },
        {
         "fieldName": "NUTS2_name",
            "operator": "==",
            "value": "Marche"   
        }
    ]
    }
    ```    
    The application return a BAD REQUEST because of the wrong syntax for the conditional operator.
    The following message is also printed if the user writes a wrong key, for example "fieldNam" instead of "fieldName" in single or multiple filter.
   
    ``` "Corpo del messaggio JSON errato" ```
- > localhost:8080/filter/

    _JSON body:_

    ```
    {
        "fieldName": "Year",
        "operator": "===",
        "value": "2000"
    }
    ```
    The application return a BAD REQUEST with the following message:
                                   
   ``` "Operatore non conforme, gli operatori ammessi sono: ==, >, >=, <, <=" ```
- > localhost:8080/statistics/Period
    
    Following the characteristic representation for the field "Programming-Period" 
    that was implemented to get statistics (and also to use filter) 
    you need to specify "PeriodStart" or "PeriodEnd".
    The following message is displayed if your request contain only "Period".   
    
    ``` "Devi specificare PeriodStart o PeriodEnd" ```
    
              
              
                     
