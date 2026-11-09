R code confidence interval generator
This code makes bar charts with 95% confidence intervals from one Excel file. It does not calculate anything: it only sorts and plots the values in the Excel file.
Excel file
Column 1: names of the departments.
Column 2: central value (rate, mean, proportion or any other parameter).
Column 3: lower limit of the confidence interval.
Column 4: upper limit of the confidence interval.
Repeat columns 2 to 4 for each of the six series, up to column 19. Column names must be the same as in lines 27 to 32 of the code.
Result
The categories appear in descending order, from the highest to the lowest value. The bar is dark blue and the confidence interval is a red line.
Before running the code
Line 14: path of the folder where the Excel file is on your computer.
Line 15: name of the Excel file.
Line 16: name of the sheet.
Then open the code in RStudio and click Source
