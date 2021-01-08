# PHPScraper and Python3 BeautifulSoup in Comparison

A while ago I found [Goutte](https://github.com/FriendsOfPHP/Goutte) by the [Symfony Team](https://symfony.com) on one of my [scouting missions on GitHub](https://github.com/spekulatius/web-stuff). It's a really neat library, assisting anyone scraping websites or simply processing some data coming from external websites. While working with Goutte for a bit, I noticed I abstracted parts of the functionality again and again. After building up a bunch of helpers I copied from project to project, I made a call to migrate these into a library. [PHPScraper](https://github.com/spekulatius/PHPScraper) was born. It was my solution to reduce the coding overhead in PHP.

I put it on GitHub, added a VuePress site as a basic documentation, and left it for the time being. Until my needs to scrape came up again. For my new project I've had to extract links with various attributes such as "target", "rel", etc. It started with some quickly put together [hacks](https://github.com/spekulatius/hacks) and turned out to be an opportunity to confirm if my scraper could hold up against more advanced libraries such as Python's [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/). I've decided to build my required functionality using my limited Python knowledge and BeautifulSoup as well as PHPScraper.

Requirements
------------

Both programs needed to fulfill these requirements:

1. run as a CLI program,
2. take an URL as argument,
3. fetch the HTML page,
4. extract all links with
   - href
   - target
   - rel
   - title
   - anchor text
6. write the result as a json to a file

The fifth part is actually simplified - the data will be processed further, but for this experiment it would be sufficient to write the link-data to the filesystem.


Scraping Links with Python 3 and BeautifulSoup 4
------------------------------------------------

As mentioned initially, my Python knowledge is limited and Google had to assist me here and there. The various package managers and strong differences between Python 2 and 3 made it more a little more work than anticipated. While the source code probably still could be optimized further, it fulfills the requirements. All in all, the source code is still simple and not too long:

```py
from bs4 import BeautifulSoup
import sys
import urllib.request
import json

# Prep the request
url = sys.argv[1]
req = urllib.request.Request(
    url,
    data=None,
    headers={
        'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0'
    }
)

# Get the site
f = urllib.request.urlopen(req)
content = f.read().decode('utf-8')

# Parse the content
soup = BeautifulSoup(content, "html.parser")

data = []
for link in soup.find_all('a'):
    data.append({
        'href': link.get('href'),
        'title': link.get('title'),
        'rel': link.get('rel'),
        'target': link.get('target'),
        'text': link.getText(),
    })

with open('./links.json', 'w') as outfile:
    json.dump(data, outfile)
```

As mentioned, it's likely not ideal.


PHPScraper Link Extractor Implementation
----------------------------------------

As mentioned above, for the PHP version, I've used my [PHPScraper library](https://phpscraper.de/). First I initialized a new composer project and then ran "composer require spekulatius/phpscraper". The source code is pretty simple:

```php
<?php

require 'vendor/autoload.php';

// Instantiate the library
$web = new \spekulatius\phpscraper();

// Navigate to the test page.
$web->go($argv[1]);

// Extract the links
$links = $web->linksWithDetails;

// Write the result
file_put_contents('./links.json', json_encode($links));
```

As the four functional lines code are commented, I guess it doesn't need many words here...

Performance Comparison: BeautifulSoup & PHPScraper
--------------------------------------------------

Comparison of the performance is naturally hard. It depends on numerous factors, of which a large chunk is out of my control. To reduce the side effects I've opted for this test setup:

 - tests are run on a dedicated server,

 - each script gets the same 30 randomly pre-selected URLs one-by-one (see here)

 - time is measured using the Linux built-in command time for the complete execution.

 - the tests are run 5 times for each script

While this isn't the best possible test setup, it should do, to get a feeling for the performance.

The test setup could be improved by e.g.,

 - handing the whole list of URLs in to reduce time required to bootstrap and interpret the scripts and

 - pre-fetching the HTML and handing it into the script to exclude network delays.

The tests were run using the following commands:

time while read url; do python3 python3-beautifulsoup-link-extractor/link-extractor.py "$url"; done < urls

and

time while read url; do php phpscraper-link-extractor/link-extractor.php "$url"; done < urls


### Performance Test Results

As mentioned above, the test is not representative and lacks fundamentals. I've still included some numbers to indicate performance differences.

The Python script with BeautifulSoup took the following time to process the url list:

 - 21.1s
 - 19.2s
 - 16.0s
 - 14.9s
 - 16.4s

The average being around 17.5 seconds to fetch and process the list of 30 URLs.

The PHP script with PHPScraper achieved the following times:

 - 12.4s
 - 12.7s
 - 10.4s
 - 12.6s
 - 12.1s

The average being around 12.0 seconds to fetch and process the list of 30 URLs. This also includes processing time for additional checks for flags such as "isNofollow", etc. which hasn't been removed from the standard library.

Other experiments and tests:

 - [Extracting the length distribution of keywords using PHPScraper](https://github.com/spekulatius/phpscraper-keyword-length-distribution-example)
 - [Extracting and filtering various keyword-sets using PHPScraper](https://github.com/spekulatius/phpscraper-keyword-scraping-example)
