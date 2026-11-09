R _code_ confidence_ interval_ generator
This code generates bar charts with confidence intervals to compare a rate across territorial units. In the temperature project, each chart shows the visit rate of the 33 territorial units of Colombia, sorted from highest to lowest. The bar represents the rate and the horizontal line, its confidence interval.
1. Requirements
Install R and RStudio. The script uses the readxl package; the figures are drawn with the basic graphics functions of R. To run it, you need a single Excel file (.xlsx) that gathers the rates and limits of the six series.
2. Obtaining and organizing the data
Extract the visit counts from the SISPRO cubes following the procedure reviewed during the training sessions. Calculate the rate for each department as visits in the 2010-2024 period over the DANE 2024 population per 10,000 inhabitants, with its 95% confidence interval (Poisson-Byar). The script does not calculate rates or intervals: it plots the values exactly as they are in the Excel file.
Prepare six series: three for visits in general and three for visits by physicians. In each group, apply the filters that correspond to the temperature category.
Category	Codes included
temperature	Codes associated with cold or heat
heat	Only codes associated with heat
cold	Only codes associated with cold
Columns in the Excel file: Rate, LL and UL (Overall and Physicians)
Each series takes up three columns of the file: the rate (Rate), the lower limit (LL) and the upper limit (UL). Record the SISPRO filters and the interval calculation method that were used in each series.
3. Format of the Excel file
The script reads a single sheet, the one indicated in the configuration (in our case, general area). Place the headers in the first row. The first column is called Department and contains the names of the 33 territorial units; the next 18 columns correspond to the rate and limits of each series. Their names start with Rate, LL or UL, followed by the group (overall or md) and the category (heat, cold or temperature), for example Rate_overall_heat. Each row corresponds to one department.
Save the rates and limits as numeric values, with no negative values and with LL less than or equal to Rate, and Rate less than or equal to UL. If you copy them from another sheet, use «Paste Values» so that the file contains the results of the formulas. Write the department names in their short form, with or without accents (for example Bogotá, San Andrés, Norte de Santander, Valle del Cauca); the script converts them to their ISO 3166-2 code and stops if a name does not match, for example Bogotá, D.C. or San Andrés, Providencia y Santa Catalina. If a department has no estimate in a series, leave its three cells (Rate, LL and UL) empty.
Abbreviated example with fictitious data:
Department	Rate_overall_heat   	LL_overall_heat	   UL_overall_heat
Antioquia	   10.52   	9.87	11.17
Bogotá	6.31	5.90	6.72
Vaupés			
Zeros are written as 0 and are plotted. Empty cells are not converted to zero: the department remains in the review table, but does not appear in that figure. If in a series only one or two of the three cells are empty, the script stops.
4. Configuration and execution
Before running the script in RStudio, adjust these parameters:
Parameter	Line	What you must enter
Input path	14	Local path of the folder that contains the Excel file
File name	15	Exact name of the file, including the .xlsx extension
Sheet	16	Exact name of the sheet that contains the data
Figure titles	22 and 23	Title of each chart; it is also the name of the output files
X-axis titles	24 to 26	Text of the horizontal axis of each chart
Columns of each series	27 to 32	Names of the rate, lower limit and upper limit columns, if they are different from those in the example
Once the parameters are updated, run the complete script; the figures are generated without typing anything in the console. If your data do not correspond to 2010-2024 or are not per 10,000 inhabitants, also change the note that appears below each figure, on line 178.
5. Output files
The script generates six individual diagrams, one for each Excel file, and a mosaic that brings the six together. The figures are exported in PDF and PNG formats. If a configured file is missing or the code detects a problem when reading it, it shows an error message.
