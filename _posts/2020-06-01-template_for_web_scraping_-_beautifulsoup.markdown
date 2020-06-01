---
layout: post
title:      "Template for Web Scraping - BeautifulSoup"
date:       2020-06-01 08:53:34 -0400
permalink:  template_for_web_scraping_-_beautifulsoup
---


Before I had much practice in it, *web-scraping* was one of the more intimidating aspects of coding. It felt like I had to leave the comfy nest of *local-host* and delve into the big and scary internet.

But like anything else, a little practice makes it seem much simpler. Much of the documentation is somewhat longwinded for an absolute beginner, so let's check out the basics.

### Web-Scraping Walkthrough

Today, let's look at a function which scrapes a url for the *Top 100 Baby Names for Boys*.

<a href='https://www.familyeducation.com/baby-names/popular-names/boys'>Here is the webpage we'll be using.</a>

And here's the working function:

```python
import requests
from bs4 import BeautifulSoup

def scrape_for_names():
    """Scrape familyeducation.com webpage for most popular boys names."""
    
    # Create a Response from requests.
		
    url = 'https://www.familyeducation.com/baby-names/popular-names/boys'
    r = requests.get(url)
    
    if not r.ok:
        return 'Failed to get a response.'
    
    # Create a 'soup' from BeautifulSoup.
		
    soup = BeautifulSoup(r.content)
    
    names_container = soup.find('section', id='block-boytopnames')

    # The names appear in two columns.
		
    # Compile a list of the two columns.
		
    columns = names_container.findAll('ul')

    # Loop through both columns and create a dictionary of rank and name.
		
    top_100_boys_names = {}
    for column in columns:
        lines = column.findAll('li')
        for line in lines:
            rank = int(line.find('strong').text.split('.')[0])
            name = line.find('a').text

            top_100_boys_names[rank] = name
    
    return top_100_boys_names
```


***


#### Let's go into detail of what's happening here.

#### 1

```python
import requests
from bs4 import BeautifulSoup
```

First of all, we import our two libraries, **requests and bs4** (actually we only need *BeautifulSoup* from bs4, so we don't need to import the whole thing).

#### 2

```python
# Create a Response from requests.

url = 'https://www.familyeducation.com/baby-names/popular-names/boys'
r = requests.get(url)
```

```requests.get(url)``` returns a Response object from our ```requests``` library. These objects store the HTML from the web pages, and some other useful information.

***

*For example:*

- `type(r)` *returns* `requests.models.Response`

- `r.status_code, r.ok` *returns* `(200, True)` *(hopefully!)*

- `r.content[:80]` *returns* `b'<!DOCTYPE html>\n<html lang="en" dir="ltr" prefix="og: 
https://ogp.me/ns#">\n    <'`

***
#### 3

```python
# Create a 'soup' from BeautifulSoup.

soup = BeautifulSoup(r.content)
```

Next we can simply take the html from `r` and create a soup using `BeautifulSoup`!

***

*For example:*

- `type(soup)` *returns*  `bs4.BeautifulSoup`

- A *BeautifulSoup* object can sift through the html quickly by its built-in methods. For example:
  - `soup.find()` will return the first instance of a tag type from the `soup`.
  - `soup.findAll()` will return a list of all the instances of a tag type from the `soup`.

***

**At this point,** we need to know the structure of the webpage in order to find the information we're looking for. We do this by right-clicking the page and selecting "Inspect".

This will pull up something like this:


<img src='https://raw.githubusercontent.com/cwf231/dsc-web-scraping-lab-onl01-dtsc-pt-041320/master/na/boysnames.PNG' width='100%'>


From this view, we can find all the tags, classes, and ids that we need.

***

#### 4

```python
# Use soup's method to find the block where the boys names are listed.

names_container = soup.find('section', id='block-boytopnames')

# The names appear in two columns.


# Compile a list of the two columns.

columns = names_container.findAll('ul')
```

You can pass in the html tag that you want (eg: `'div'`, `'ul'`, `'h1'`, etc) as well as the `id` or `class` to pull out the specific sections of the page you're looking for.

- In this case, we select `names_container` which is the largest section of the page that all the names are listed in.

- There are two columns within the `names_container`. We can use `BeautifulSoup.findAll()` to return a list of all the `ul` tags in `names_container`.

***

#### 5


**At this point**, all that's left is to loop through each column and get all the names from another `BeautifulSoup.findAll()`. I decided to save them into a dictionary for this example.

```python
top_100_boys_names = {}
for column in columns:
    lines = column.findAll('li')
```

`lines` returns a list of `li` tags, as before.

```
    for line in lines:
        rank = int(line.find('strong').text.split('.')[0])
        name = line.find('a').text
				
				top_100_boys_names[rank] = name
```

***

Each line is simple:
```
<li>
    <strong>1.</strong>
    <a href="https://www.familyeducation.com/baby-names/name-meaning/liam">Liam</a>
</li>
```

- `line.find('strong')` returns the whole `strong` section.
- `line.find('strong').text` returns the text only.
- `line.find('strong').text.split('.')[0]` returns only the selection up to the `'.'` (just the number).



*Sifting through the `<a>` is essentially the same. We can pull out the text with `line.find('a').text`*

***
#### 6
#### And...we're done!

`top_100_boys_names`

returns

*```
{1: 'Liam',
 2: 'Noah',
 3: 'William',
 4: 'James',
 5: 'Oliver',
 . . .
 }
 ```*
 
etc...

***

BeautifulSoup is an awesome tool which makes finding information on websites extremely easy!


