```python3
from selenium import webdriver
import xlrd
import xlwt
import pandas as pd

#variables, fill in with stuff
usernameStr = 'putYourUsernameHere'
passwordStr = 'putYourPasswordHere'
browser = webdriver.Chrome()
browser.get(('https://hts.symbionhealth.com/shop/'))
pointer = 0

#opens workbook with barcodes and makes a list of them, fill in with stuff
df = pd.read_excel('filename.xlsm', sheetname=0) # can also index sheet by name or fetch all sheets
Barcodes = df['column name'].tolist()
length = len(Barcodes)

# fill in username and password and sign in
username = browser.find_element_by_id('UserName')
username.send_keys(usernameStr)
password = browser.find_element_by_id('Password')
password.send_keys(passwordStr)
signInButton = browser.find_element_by_id('ctl00_cplMain_Login2_LoginButton')
signInButton.click()

#go to barcode page, fill in with ids
navigateToOrdering = browser.find_element_by_id('')
navigateToOrdering.click()
stocksearcher = browser.find_element_by_id('')
stocksearcherbutton = browser.find_element_by_id('')

site_stock = browser.find_element_by_id('')
site_barcode = browser.find_element_by_id('')
site_name = browser.find_element_by_id('')
list_to_pass = ['product1value' - 'product2value']
listRange = len(list_to_pass)
for range in length:
	stocksearcher.send_keys(Barcodes[pointer])
	stocksearcherbutton.click()
	pointer =+1
		for range in listRange:
		stock = site_stock.get_attribute('title')
		barcode = site_barcode.get_attribute('title')
		name = site_name.get_attribute('title')
```
