Extracting Data from XML

In this assignment you will write a Python program somewhat similar to http://www.py4e.com/code3/xml3.py. The program will prompt for a URL, read the XML data from that URL using urllib and then parse and extract the comment counts from the XML data, compute the sum of the numbers in the file.

We provide two files for this assignment. One is a sample file where we give you the sum for your testing and the other is the actual data you need to process for the assignment.

Sample data: http://py4e-data.dr-chuck.net/comments_42.xml (Sum=2553)
Actual data: http://py4e-data.dr-chuck.net/comments_2212606.xml (Sum ends with 85)
You do not need to save these files to your folder since your program will read the data directly from the URL. Note: Each student will have a distinct data url for the assignment - so only use your own data url for analysis.

```python
# Python libraries needed for this exercise
import urllib.request
import xml.etree.ElementTree as ET

# Lazy code for cmd so you don't have to keep inputting url
x = int(input('Enter location: '))
if x < 1 : 
    url = 'http://py4e-data.dr-chuck.net/comments_42.xml'
else: 
    url = 'http://py4e-data.dr-chuck.net/comments_2212606.xml'

# Coverting XML data to python    
print('Retrieving', url)
uh = urllib.request.urlopen(url).read()
print('Retrieved',len(uh),'characters')
tree = ET.fromstring(uh)

# Summing the requirement
b = 0
counts = tree.findall('.//count')
nums = list()
for result in counts:
    
    # print(result.text)
    a = int(result.text)
    b = a + b
    
print('Total sum: ',b)
```
