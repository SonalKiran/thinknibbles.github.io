---
layout: post
title:  "Python Multithreaded Web Image Scraper"
date:   2021-02-02 01:35:00 +0800
category: Python
categories: [Python]
tags: [Python, ImageScraper, Multithreading]
---
### Who is this post for ?

Software developers / machine learning engineers / data scients looking for a python script to automate downloading of images from the web. 

### Tools this post will be using : 

1. Python 3
2. Chrome driver
3. MacOS (Catalina)

### Let's start :

Data is not always readily available in a form in which it can be consumed. Good data even less so. Data scientists are all too aware of this issue. In recent years, especially due to the growth in the field of data science, data gathering has become an important part of the lives of data scientists the world over. This post is written with the aim of helping others save time when it comes to downloading images from the web.

### Source Code :

If you like, the source code for this entire post is available [here](https://github.com/SonalKiran/PyImageScraper) and can be downloaded for free.

This script is divided into three parts - 

1. A function to fetch URLs:  This function takes the following arguments - 

   1. Keyword for which images are to be downloaded
   2. Number of URLs to be fetched for that keyword 

   Given the above arguments, this function then saves the keyword and its URLs in a dictionary named 'master_urls'.

2. A function to fetch images: This function takes the following arguments -

   1. Request.session() object which allows the persistence of parameters across requests
   2. URL from which to download image
   3. Filename under which to store the downloaded image

   Given the above arguments, this function then attempts to download the image and save it under the given filename. If instead, we have the base64 of the image, it converts and saves that base64 as an image.

3. The main function which employs multithreading to speed up the entire process: It is here that all the heavy lifting is done. 
   1. We first read the keywords from the 'queries.csv'  and save them in the 'keywords' list.
   2. Since fetching URLs from a webpage is an IO-bound task (most of the time is spent waiting on input-output operations), multithreading helps to significantly reduce the time taken by this activity.  We use concurrent.futures module's [`ThreadPoolExecutor`](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.ThreadPoolExecutor) for this. Please refer to the linked documentation if you wish to understand how this executor works. All the keyword-URL pairs are stored in the 'master_urls' dictionary.
   3. We then save this dictionary as 'keyword_urls.txt' in case we ever need to reference it later. To load this dictionary from a text file, the required code has been provided, although it has been commented out since we won't be requiring it now.
   4. Having curated a list of URLs for each keyword, we then pass this list on to the 'fetch_img' function. Again, since this is mostly an IO-bound task, we employ multithreading to speed things up. Images for each keyword are stored under a folder by the same name inside the 'images' folder in the root folder. 

```python
# imports
from selenium import webdriver
import time, requests
import pandas as pd
import os
import concurrent.futures
import base64
import ast
from selenium.common.exceptions import WebDriverException
from selenium.common.exceptions import TimeoutException

# function to fetch urls
def fetch_url(search_query, url_count = 20):
    browser = webdriver.Chrome()
    search_url = f"https://www.google.com/search?site=&tbm=isch&source=hp&biw=1873&bih=990&q={search_query}"
    images_url = []

    # open browser and begin search
    browser.get(search_url)
    try:
        elements = browser.find_elements_by_class_name('rg_i')
    except TimeoutException:
        print("Loading took too much time!")

    count = 0
    for e in elements:
        # fetch image urls
        try:
            e.click()
            time.sleep(1.5)
            element = browser.find_elements_by_class_name('v4dQwb')
            # Google Chrome logic
            if count == 0:
                big_img = element[0].find_element_by_class_name('n3VNCb')
                count += 1
            else:
                big_img = element[1].find_element_by_class_name('n3VNCb')
                count += 1
            images_url.append(big_img.get_attribute("src"))
        except WebDriverException as e:
            print(f"Web Driver Exception Occurred: {e.__str__()}")
            continue

        # save urls for the given search query in a dictionary
        if len(images_url) == url_count:
            global master_urls
            master_urls[search_query] = images_url
            browser.quit()
            break

# function to fetch images
def fetch_img(sess, url, filename):
    if 'base64' in url:
        try:
            b64 = url[url.index('base64')+6:]
            with open(filename, 'wb') as f:
                f.write(base64.b64decode(b64))
        except Exception as e:
            print(f"Base64 to Image - Exception Occurred: {e.__str__()}")
    else:
        try:
            r = sess.get(url, stream = True, timeout=3)
            if r.status_code == 200:
                # Open a local file with wb ( write binary ) permission.
                with open(filename, 'wb') as fd:
                    for chunk in r.iter_content(chunk_size=128):
                        fd.write(chunk)
                print(f'Image successfully downloaded: {filename}')
            else:
                print(f'Unable to downloaded image: {filename}')
        except Exception as e:
            print(f"Exception Occurred: {e.__str__()}")

# retrieve master_urls dict from keyword_urls.txt
# with open('./keyword_urls.txt', 'r') as file:
#     new_dict = file.read()
# master_urls = ast.literal_eval(new_dict)

# define the path of csv containing keywords
csv_path = './queries.csv'

# set destination directory to store images
dest_dir = './images'

# initialize dict
master_urls = {}

def main():
    # load search queries from csv
    data = pd.read_csv(csv_path, header=None)
    data.columns = ["Query"]

    # save queries to a list
    keywords = [data["Query"][i] for i in range(len(data))]

    # fetch urls using multithreading and save to master_urls dict
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as exec:
        future = [exec.submit(fetch_url, search_query=val, url_count=10) for
                  val in keywords]

    #  ensure dest_dir exists
    if not os.path.exists(dest_dir):
        os.mkdir(dest_dir)

    # save master_urls in text file for future reference
    with open(os.path.join(dest_dir,'keyword_urls.txt'), 'w') as file:
        file.write(str(master_urls))

    for key, values in master_urls.items():
        # create directory for every keyword for which image is to be downloaded
        if not os.path.exists(os.path.join(dest_dir, key)):
            os.mkdir(os.path.join(dest_dir, key))
        # download images
        with requests.Session() as sess:
            with concurrent.futures.ThreadPoolExecutor(max_workers=10) as exec:
                future = [exec.submit(fetch_img, sess=sess, url=val, filename=os.path.join(dest_dir, key, str(i) + '.jpg')) for i, val in enumerate(
                    values)]

main()
```



**I hope this post was helpful.**

**Cheers!**

