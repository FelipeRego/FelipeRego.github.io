---
layout: post
title:  "Creating Fake (Mock) Data with Python"
date: 2019-01-11
excerpt: "Mocking up data for analytics, datawarehouse or unit test can be challenging. This Python package is a fast and easy way to generate fake (mock) data."
image: "/images/fakedata1.jpg"
permalink: /blog/2019/01/11/Creating-Fake-Mock-Data-Python
---

{% include advertisements.html %}

In this post I wanted to share an interesting Python package and some examples I found while helping a client build a prototype. The package generates mock or fake data and is simple to use and works well if you need to quickly generate some dummy or mock data for testing purposes.

From my perspective, it can also be useful for teaching analytics and also if you want to generate fake or mock up data with many different data types easily and quickly. You can then export it as a .csv or use pandas data frames for other data science and analytics use cases. 

It contains some great documentation too: [Faker's documentation](https://faker.readthedocs.io/en/stable/index.html)




A few examples I wanted to share are shown below. Start by importing the Faker library and pandas:

```python
from faker import Faker
import pandas as pd
```


Here we initialise Faker generator and create an example of generating a fake data for a random name:

```python
faker = Faker()
faker.name()
```




    'Eric Poole'


You'd probably want to generate more than one fake data record at a time:

```python
for n in range(5):
    print(faker.name())
```

    Whitney Davies
    Christopher Johnson
    Robert Washington
    Zachary Williams
    Mark Ramirez



There are many other different types of fake data you can generate by using 'providers'. For a list of providers see the docs.
```python
from faker.providers import internet, geo
```



You can generate fake mock data on latitude and longitude, for example:

```python
for n in range(5):
    print(faker.local_latlng(country_code="AU", coords_only=False))
```

    ('-31.95224', '115.8614', 'Perth', 'AU', 'Australia/Perth')
    ('-37.88333', '145.06667', 'Carnegie', 'AU', 'Australia/Melbourne')
    ('-32.05251', '115.88782', 'Willetton', 'AU', 'Australia/Perth')
    ('-32.05251', '115.88782', 'Willetton', 'AU', 'Australia/Perth')
    ('-33.75881', '150.99292', 'Baulkham Hills', 'AU', 'Australia/Sydney')




And if you wanted to generate a custom list of fake data types and create a pandas data frame with these fake data points:

```python
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




There is also an option to bring fake data about a profile pre-built by the package:

```python
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


And if we wanted to - again - create a pandas data frame from this same mock fake data:



```python
df2 = []

for n in range(3):
    df2.append(list(faker.profile().values()))

df2 = pd.DataFrame(df2, columns=faker.profile().keys())
df2
```



There are many other options you can use to generate other fake data and also to tweak how some of the properties are generated. Again, the [docs](https://faker.readthedocs.io/en/stable/index.html) are very helpful indeed!
