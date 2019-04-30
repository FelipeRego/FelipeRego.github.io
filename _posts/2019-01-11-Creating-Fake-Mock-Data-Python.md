---
layout: post
title:  "Creating Fake (Mock) Data with Python"
date: 2019-01-11
excerpt: "Mocking up data for analytics, datawarehouse or unit test can be challening. This Python package is a fast and easy way to generate fake data."
image: "/images/fakedata1.jpg"
permalink: /blog/2019/01/11/Creating-Fake-Mock-Data-Python
---



{% include advertisements.html %}


In this post I wanted to share an interesting Python package and some examples I found while helping a client build a prototype. The package generates mock or fake data and is simple to use and works well if you need to quickly generate some dummy or mock data for testing purposes. From my perspective, it can also be useful for teaching analytics and also if you want to generate fake or mock up data with many different data types easily and quickly. You can then export it as a .csv or use pandas data frames for other data science and analytics use cases. 

It contains some great documentation too: [Faker's documentation](https://faker.readthedocs.io/en/stable/index.html)


{% include advertisements.html %}


A few examples I wanted to share are shown below:


```python
# import libraries
from faker import Faker
import pandas as pd
```


```python
# initialise Faker generator
faker = Faker()

# example of generating a fake data for a name
faker.name()
```




    'Eric Poole'




```python
# you'd probably want to generate more than one record at a time
for n in range(5):
    print(faker.name())
```

    Whitney Davies
    Christopher Johnson
    Robert Washington
    Zachary Williams
    Mark Ramirez



```python
# there are many other different types of fake data you can generate
# by using 'providers'
from faker.providers import internet, geo

# for a list of providers see the docs
```


```python
# you can generate fake mock data about latitude and longitude for example
for n in range(5):
    print(faker.local_latlng(country_code="AU", coords_only=False))
```

    ('-31.95224', '115.8614', 'Perth', 'AU', 'Australia/Perth')
    ('-37.88333', '145.06667', 'Carnegie', 'AU', 'Australia/Melbourne')
    ('-32.05251', '115.88782', 'Willetton', 'AU', 'Australia/Perth')
    ('-32.05251', '115.88782', 'Willetton', 'AU', 'Australia/Perth')
    ('-33.75881', '150.99292', 'Baulkham Hills', 'AU', 'Australia/Sydney')



```python
# if you wanted to generate a custom list of fake data types
# and create a pandas data frame with these fake data points

df = []

for n in range(5):
    df.append({'Lat': faker.coordinate(center=74.0, radius=0.10),
               'Lon': faker.coordinate(center=40.8, radius=0.10),
               'Txt': faker.sentence(),
               'Nam': faker.name(),
               'Add': faker.address(),
               'Job': faker.job()
              })

df = pd.DataFrame(df)
df = df[['Nam', 'Job', 'Txt', 'Add', 'Lat', 'Lon']]
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Nam</th>
      <th>Job</th>
      <th>Txt</th>
      <th>Add</th>
      <th>Lat</th>
      <th>Lon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Joe Taylor</td>
      <td>Doctor, hospital</td>
      <td>How room deep less moment leader civil your.</td>
      <td>PSC 7330, Box 9160\nAPO AA 42582</td>
      <td>73.975584</td>
      <td>40.825469</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Laura Roach</td>
      <td>Administrator, sports</td>
      <td>Plant financial forward city forget.</td>
      <td>94636 Travis Avenue Suite 621\nPort Brandonmou...</td>
      <td>74.015006</td>
      <td>40.890276</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Terry Anderson</td>
      <td>Advertising art director</td>
      <td>Course how direction election reveal stuff str...</td>
      <td>0791 Diane Groves\nAndrewchester, IA 98686</td>
      <td>73.919303</td>
      <td>40.848450</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mckenzie Taylor</td>
      <td>Animal nutritionist</td>
      <td>Although fly high yeah data common campaign.</td>
      <td>381 Calvin Highway Suite 109\nHallville, MS 08169</td>
      <td>73.901170</td>
      <td>40.742034</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Christine Harper</td>
      <td>Engineer, materials</td>
      <td>Push home arm.</td>
      <td>5162 Miller Village Suite 046\nWest Marvinches...</td>
      <td>73.981773</td>
      <td>40.711712</td>
    </tr>
  </tbody>
</table>
</div>




```python
# there is also an option to bring fake data about a profile
faker.profile()
```




    {'job': 'Architect',
     'company': 'Martinez, Cruz and West',
     'ssn': '311-64-2980',
     'residence': '3864 Sanford Dam Suite 803\nWest Leroy, MS 46211',
     'current_location': (Decimal('-38.669420'), Decimal('86.843925')),
     'blood_group': 'B+',
     'website': ['http://church.com/',
      'https://www.mayer-maldonado.com/',
      'http://matthews.org/'],
     'username': 'ocannon',
     'name': 'David Johnson',
     'sex': 'M',
     'address': '1431 Berry Extensions\nSouth Jameshaven, ME 13555',
     'mail': 'wilkinssteven@gmail.com',
     'birthdate': datetime.date(1947, 1, 10)}




```python
# and if we wanted to - again - create a pandas data frame
# from this same mock fake data
df2 = []

for n in range(3):
    df2.append(list(faker.profile().values()))

df2 = pd.DataFrame(df2, columns=faker.profile().keys())
df2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>job</th>
      <th>company</th>
      <th>ssn</th>
      <th>residence</th>
      <th>current_location</th>
      <th>blood_group</th>
      <th>website</th>
      <th>username</th>
      <th>name</th>
      <th>sex</th>
      <th>address</th>
      <th>mail</th>
      <th>birthdate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Museum/gallery exhibitions officer</td>
      <td>Steele-Miller</td>
      <td>105-57-8308</td>
      <td>PSC 9220, Box 3646\nAPO AA 56745</td>
      <td>(-35.690955, 5.909455)</td>
      <td>A+</td>
      <td>[http://www.johnson-christian.biz/, http://bro...</td>
      <td>savannah85</td>
      <td>Nicole Lewis</td>
      <td>F</td>
      <td>USCGC Rivera\nFPO AA 64485</td>
      <td>hollandchristopher@gmail.com</td>
      <td>1931-07-02</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Engineer, automotive</td>
      <td>Thomas-Shaffer</td>
      <td>208-82-7087</td>
      <td>55025 Brett Cliffs Suite 663\nWest Williammout...</td>
      <td>(70.2920835, 72.574494)</td>
      <td>AB+</td>
      <td>[https://davis.org/, https://www.nguyen.com/]</td>
      <td>ashley68</td>
      <td>Amy Morgan</td>
      <td>F</td>
      <td>7618 James Overpass\nMendozaside, OR 74154</td>
      <td>brianmann@gmail.com</td>
      <td>1903-10-17</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Counselling psychologist</td>
      <td>Evans Inc</td>
      <td>702-05-1171</td>
      <td>9210 Jeremy Expressway\nSouth Ashleyfort, IL 1...</td>
      <td>(-73.112756, -153.906283)</td>
      <td>AB+</td>
      <td>[http://www.zimmerman.com/, http://rocha.com/]</td>
      <td>elliottcharles</td>
      <td>Pamela Hicks</td>
      <td>F</td>
      <td>395 Turner Parkway\nWallermouth, GA 04676</td>
      <td>ygutierrez@hotmail.com</td>
      <td>1915-07-19</td>
    </tr>
  </tbody>
</table>
</div>



There are many other options you can use to generate other fake data and also to tweak how some of the properties are generated. Again, the [docs](https://faker.readthedocs.io/en/stable/index.html) are very helpful indeed!
