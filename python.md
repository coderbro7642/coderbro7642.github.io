# Ryan Mitchell's Python Portfolio
[Home](index.md)    [Python Portfolio](python.md)   [R and spatial modelling Portfolio](R.md)   [Resume](resume.md)

The following block of code is a webcrawler to log in to a website and then search the website using a list of barcodes it's extracted from an excel file to return stock levels on the website, this was built using the pycharm code environment and chrome webdriver. It ran at an efficiency of 1200 barcodes per hour.
Note: Selenium was used as the original website this was constructed for used dynamic javascript data from an .aspx extension.
```python3
from selenium import webdriver
import time
from openpyxl import load_workbook
from openpyxl import Workbook

# variables, fill in with stuff
usernameStr = 'putusernamehere'
passwordStr = 'putpasswordhere'
Barcode_list = []
list_of_barcodes = []
raw_data = []

#opens list of barcodes and inputs to Barcode_list
workbook = load_workbook(filename="input.xlsx", read_only=True)
sheet = workbook.active
for row in sheet.iter_rows(min_row=0,max_row=, values_only=True):
    value = row[0]
    Barcode_list.append(value)

#opens webpage, signs in
driver = webdriver.Chrome(executable_path='locationofchromedriver')
driver.get("websitename")
username = driver.find_element_by_name("nameofusernameinput")
username.send_keys(usernameStr)
password = driver.find_element_by_name("nameofpasswordinput")
password.send_keys(passwordStr)
signInButton = driver.find_element_by_id('idofsigninbutton')
signInButton.click()

#select advanced search
advancedSearch = driver.find_element_by_id('btnAdvancedSearch')
advancedSearch.click()

#fill in barcodes with ids
for barcode in Barcode_list:
    raw_data.clear()
    barcodesearcher = driver.find_element_by_name('nameofbarcodeinput')
    barcodesearcher.clear()
    barcodesearcher.send_keys(barcode)
    barcodesearcherbutton = driver.find_element_by_id('idofproductsearchbutton')
    barcodesearcherbutton.click()
    time.sleep(2)
    Barcodeselector = driver.find_elements_by_id('idofbarcode')
    Nameselector = driver.find_elements_by_id('idofproductname')
    Stockselector = driver.find_elements_by_class_name('classnameofstockvalue')
    for i in Barcodeselector:
        if i.get_attribute("textContent"):
            raw_data.append(i.get_attribute("textContent"))
    for i in Nameselector:
        if i.get_attribute("textContent"):
            raw_data.append(i.get_attribute("textContent"))
    for i in Stockselector:
        if i.get_attribute("textContent"):
            raw_data.append(i.get_attribute("textContent"))
    converteddata = list(raw_data)
    print(raw_data)
    list_of_barcodes.append(converteddata)
    workbook_name = 'output1.xlsx'
    wb = Workbook()
    page = wb.active
    for info in list_of_barcodes:
        page.append(info)
    wb.save('output1.xlsx')
    time.sleep(1)
driver.quit()
```
