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

There is some wrong field and therefore some data is managed in a dedicated way in order to be corrected. Then follow these manual manipulation operations on the CSV data:
#inserire qua codice che gestisce regex e rimpiazza con lo zero i campi vuoti!
