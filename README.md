# Object Oriented Programming Project
The objective of our project was the creation of a JAVA application that has as its main purpose the creation of a set of classes capable of modeling a specific assigned data-set into objects.
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
