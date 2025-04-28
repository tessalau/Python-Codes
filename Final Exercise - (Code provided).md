Retrieving GEOData
Download the code from www.py4e.com/code3/opengeo.zip - then unzip the file and edit where.data to add an address nearby where you live - don't reveal where you live. Then run the geoload.py to lookup all of the entries in where.data (including the new one) and produce the geodata.sqlite. Then run geodump.py to read the database and produce where.js. You can run the programs and then scroll back to take your screen shots when the program finishes. Then open where.html to visualize the map. Take screen shots as described below. Make sure that your added location shows in all three of your screen shots.


This is a relatively simple assignment. Don't take off points for little mistakes. If they seem to have done the assignment give them full credit. Feel free to make suggestions if there are small mistakes. Please keep your comments positive and useful. If you do not take grading seriously, the instructors may delete your response and you will lose points.
Assignment specification:

GEOLOAD.PY
```python
import urllib.request, urllib.parse, urllib.error
import http
import sqlite3
import json
import time
import ssl
import sys

# https://py4e-data.dr-chuck.net/opengeo?q=Ann+Arbor%2C+MI
serviceurl = 'https://py4e-data.dr-chuck.net/opengeo?'

# Additional detail for urllib
# http.client.HTTPConnection.debuglevel = 1

conn = sqlite3.connect('opengeo.sqlite')
cur = conn.cursor()

cur.execute('''
CREATE TABLE IF NOT EXISTS Locations (address TEXT, geodata TEXT)''')

# Ignore SSL certificate errors
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

fh = open("where.data")
count = 0
nofound = 0
for line in fh:
    if count > 100 :
        print('Retrieved 100 locations, restart to retrieve more')
        break
    
    # Address is cleaned from line.strip() in proper format and prints a blank line, 
    # Encode Address to UTF8 and then wrap it using memoryview() for efficient binary data handling
    # Checks the DB for if the address exists.If found, it prints Found msg, skips rest of code and restarts the loop
    # If not, it pass (DO Nothing) to next step
    address = line.strip()
    print('')
    cur.execute("SELECT geodata FROM Locations WHERE address= ?",
        (memoryview(address.encode()), ))

    try:
        data = cur.fetchone()[0]
        print("Found in database", address)
        continue
    except:
        pass
    
    # The new Address gets parsed into url format to search in opengeo and retrieve result
    # When a 100 of these is done, this program ends and needs to be restarted
    parms = dict()
    parms['q'] = address

    url = serviceurl + urllib.parse.urlencode(parms)

    print('Retrieving', url)
    uh = urllib.request.urlopen(url, context=ctx)
    data = uh.read().decode()
    print('Retrieved', len(data), 'characters', data[:20].replace('\n', ' '))
    count = count + 1

    try:
        js = json.loads(data)
    except:
        print(data)  # If error, error gets printed, skip rest of code, and restart loop
        continue
    
    # If no error, sanity check if result of Address is what we are loooking of
    # If not, then it adds to noFound count which is printed at the end of the entire code
    if not js or 'features' not in js:
        print('==== Download error ===')
        print(data)
        break

    if len(js['features']) == 0:
        print('==== Object not found ====')
        nofound = nofound + 1
    
    # If sanity check ok, insert new Address and result into sql db
    cur.execute('''INSERT INTO Locations (address, geodata)
        VALUES ( ?, ? )''',
        (memoryview(address.encode()), memoryview(data.encode()) ) )

    conn.commit()
    
    # If 10 new Address has been added, it pauses for 5s before continuing the 100 loops
    if count % 10 == 0 :
        print('Pausing for a bit...')
        time.sleep(5)

if nofound > 0:
    print('Number of features for which the location could not be found:', nofound)

print("Run geodump.py to read the data from the database so you can vizualize it on a map.")

```
GEODUMP.PY

``` python
import sqlite3
import json
import codecs

conn = sqlite3.connect('opengeo.sqlite')
cur = conn.cursor()

# Open where.js in write mode with UTF-8 encoding via codecs.open
# Select ALL entries from LOcation table in the opened SQL database 
cur.execute('SELECT * FROM Locations')
fhand = codecs.open('where.js', 'w', "utf-8")
fhand.write("myData = [\n")
count = 0

# For each selected row, decode the row and store in 'data' as STRING
# Parse the data with json library into python readables
# Sanity check if 'features' exist, if not then reloop

for row in cur :
    data = str(row[1].decode())
    try: js = json.loads(str(data))
    except: continue

    if len(js['features']) == 0: continue
    
# If pass sanity check, retrieve the lat, lng, where, and replace single quotes with double quotes in 'where'
# If error, then print unexpected format and the js line that causes the error
    try:
        lat = js['features'][0]['geometry']['coordinates'][1]
        lng = js['features'][0]['geometry']['coordinates'][0]
        where = js['features'][0]['properties']['display_name']
        where = where.replace("'", "")
    except:
        print('Unexpected format')
        print(js)

    try :
        print(where, lat, lng)
# If success, enter a new line and convert lat lng where into string and print, If error, reloop
        count = count + 1
        if count > 1 : fhand.write(",\n")
        output = "["+str(lat)+","+str(lng)+", '"+where+"']"
        fhand.write(output)
    except:
        continue

fhand.write("\n];\n")
cur.close()
fhand.close()
print(count, "records written to where.js")
print("Open where.html to view the data in a browser")

```

