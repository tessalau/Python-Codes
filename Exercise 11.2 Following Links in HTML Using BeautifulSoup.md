Following Links in Python

In this assignment you will write a Python program that expands on http://www.py4e.com/code3/urllinks.py. The program will use urllib to read the HTML from the data files below, extract the href= vaues from the anchor tags, scan for a tag that is in a particular position relative to the first name in the list, follow that link and repeat the process a number of times and report the last name you find.

We provide two files for this assignment. One is a sample file where we give you the name for your testing and the other is the actual data you need to process for the assignment

Sample problem: Start at http://py4e-data.dr-chuck.net/known_by_Fikret.html
Find the link at position 3 (the first name is 1). Follow that link. Repeat this process 4 times. The answer is the last name that you retrieve.
Sequence of names: Fikret Montgomery Mhairade Butchi Anayah
Last name in sequence: Anayah
Actual problem: Start at: http://py4e-data.dr-chuck.net/known_by_Cassidy.html
Find the link at position 18 (the first name is 1). Follow that link. Repeat this process 7 times. The answer is the last name that you retrieve.
Hint: The first character of the name of the last page that you will load is: K
Strategy
The web pages tweak the height between the links and hide the page after a few seconds to make it difficult for you to do the assignment without writing a Python program. But frankly with a little effort and patience you can overcome these attempts to make it a little harder to complete the assignment without writing a Python program. But that is not the point. The point is to write a clever Python program to solve the program.

```python

import urllib.request, urllib.parse, urllib.error
from bs4 import BeautifulSoup
import ssl
import re

# Ignore SSL certificate errors
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter website: ')
html = urllib.request.urlopen(url, context=ctx).read()

repeat = int(input('Enter the repetition: '))
pos = int(input('Enter the position: '))-1

x = re.findall('by_(\S*).html',url)

listname = list()
listlink = list()
finalnamelist=list()

html = urllib.request.urlopen(url, context=ctx).read()

finalnamelist.append(x)

while repeat > 0 :
    soup = BeautifulSoup(html, 'html.parser')
    tags = soup('a')
    for tag in tags:
        listname.append(tag.contents[0])
        listlink.append(tag.get('href',None)) 
    finalnamelist.append(listname[pos])
    print(listlink[pos])
    html = urllib.request.urlopen(listlink[pos], context=ctx).read()
    listlink.clear()
    listname.clear()
    repeat = repeat - 1
print(finalnamelist)
```
