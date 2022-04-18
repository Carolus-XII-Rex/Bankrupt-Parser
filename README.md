# Bankruptcy register parser
Parser for bankruptcy register website: https://old.bankrot.fedresurs.ru/

The parsing algorithm written to solve a real-life task: a list of individuals has to be checked for being in the register of bankruptcy. And if one is in the register, certain data relating to him must be extracted.

> As the register is publicly available all the data relating to individuals and entities is NOT considered as personal data

## Tasking


Each individual is represented via Tax Identification Number (TIN). We will consider a few of those to demonstrate the work of the algorithm. 

The data to be extracted:

1. Information relating to the individual:
- Last name (*Фамилия*)
- First name (*Имя*)
- Middle name (*Отчество*)
- Birth date (*Дата рождения*)
- Birth place (*Место рождения*)
- Bankruptcy region (*Регион ведения дела о банкротстве*)
- Primary State Registration Number of Individual Entrepreneur (*ОГРНИП*)
- Insurance number of the individual ledger account (*СНИЛС*)
- Previous names (*Ранее имевшиеся ФИО*)
- Debtor category (*Категория должника*)
- Address (*Место жительства*)
- Additional information (*Дополнительная информация*)  
![PrivatePersonCard_Info](/Screenshots/PrivatePersonCard_Info.png)  

2. The date and the name of the last document, which name contains "*заверш*" (From the "KAD documents" (*Документы КАД*) tab)  
![PrivatePersonCard_KAD](/Screenshots/PrivatePersonCard_KAD.png)  
Further items are to be extracted from the last arbitration manager report (*Финальный отчет*) (From the "Arbitration managers' reports" (*Отчеты АУ*) tab)  
![PrivatePersonCard_FinalReport](/Screenshots/PrivatePersonCard_FinalReport.png)

3. The opening (*Дата начала*) and the closing (*Дата окончания*) dates of the bankruptcy procedure  
![FinalReport_1](/Screenshots/FinalReport_1.png)

4. The amount of satisfied claims (*Сумма удовлетворенных требований*)  
![FinalReport_2](/Screenshots/FinalReport_2.png)

5. The release of the citizen from obligations (*Освобождение гражданина от обязателсьтв*)  

6. The completion of bankruptcy status (*Произвдство по делу о банкротстве*)   
 
7. The name of the attached document (*Прикрепленные файлы*)  
![FinalReport_3](/Screenshots/FinalReport_3.png)    


## Algorithm description


The algorithm gets the list of TINs as the input ([data/TINs.xlsx](data/TINs.xlsx)) and produces a pandas DataFrame with the results as the output (writing it to [output/BankruptInformation.xlsx](output/BankruptInformation.xlsx)). It is also possible to use the name of the individual as the input([*](#get_headers_for_names)).

If an individual with a certain TIN is not on the register, the corresponding row in the table will consist of "(No data)" (*н/д*) fields. If an individual is presented in the register, but a particular item of information required is absent, then "(No data)" (*н/д*) is returned in the corresponding field.

The TIN is passed into the
```{python}
get_headers_for_TIN(TIN) 
```
function which forms the headers for all the requests allowing to navigate through the required pages.  

It is also possible to use the 
```{python}
get_headers_for_names(last_name, first_name, middle_name)
```
function <a name="get_headers_for_names">(*)</a> and form the headers from the names instead of TIN.

Using *requests*  and *Beautiful Soup* libraries the formatted text of the response is obtained. Then, using *re* library and html tags the search is conducted. (See the block 'Functions to extract the data required')

An important point to mention is the
```{python}
find_string(reqired_string, list_of_strings)
```
function. After looking through a certian number of individuals manually, it became clear that there are lots of typos in the required pages. Thus searching for some exact phrases may not be successfull. This function allows to search for a string assuming that a certain number of typos might have taken place. (See [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance))

In the end the whole process of data extraction is placed into cycle, iterating through TINs.






