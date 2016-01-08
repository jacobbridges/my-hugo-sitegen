+++
baseurl = "http://jacobbridges.github.io/"
date = "2015-07-01T00:06:10-04:00"
draft = false
title = "web scraping, threads, and queues"
tags = ["python"]
categories = ["post"]

+++

### The Problem

```python
urls = get_urls()  # len(urls) > 5000
for url in urls:
    r = requests.get(url)
    page = pyquery(r.text)
    data = page("#data").text()
    # do something with data
```

Look familiar? This was my normal "quick fix" web scraping script in Python. You may prefer BeautifulSoup to PyQuery, but from all the StackOverflow questions I have read this bit of "quick fix" scraper code is very common.

The problem is that code will take almost 4 hours to complete. By continuing to publish code like this, we are furthering the stereotype that Python is slow. By using a few extra lines of code, I will show you how to take that 4 hours and cut it down to less than 5 minutes.

### Thread It

```python
from threading import Thread
```

The [threading module](https://docs.python.org/2/library/threading.html) is a deep well of power--if you are not familiar with it I suggest you go to the extrenal references section at the bottom of this post. There are some great  posts, my favorite of these being the article by Doug Hellmann at [PyMOTW](http://pymotw.com/2/about.html).

Let's create a simple scraper worker function:

```python
def scraper_worker(urls):
    for url in urls:
        r = requests.get(url)
        page = pyquery(r.text)
        data = page("#data").text()
        # do something with data
```

So we send our new function a list of urls, it scrapes them. Ez-Pz.

Now that we have a worker function, let's create several "scraper workers" with the threading module.

```python
# Create 5 scraper workers
for i in range(5):
    t = Thread(target=scraper_worker, args=(urls, ))
    t.start()
```

See a problem with this? If we start 5 scraper workers and give all of them the full list of urls we want to scrape, each page will be scraped 5 times!

One common way to fix this problem is by slicing the url list into chunks and feeding a different chunk to each worker. The workers will effectively scrape all the pages in 1/5 the original time--but there is a better way. A more efficient way. 

### Queue It

```python
from Queue import Queue
```

The Queue module has several built in queue data types to play with, but for this project I used the standard FIFO Queue. 

In Python, queues are thread safe. So we can push all the urls into the queue and share it with all the worker threads.

```python
def scraper_worker(q):
    while not q.empty():
        url = q.get()
        r = requests.get(url)
        page = pyquery(r.text)
        data = page("#data").text()
        # do something with data
        q.task_done()

# Create a queue and fill it
q = Queue()
map(q.put, urls)

# Create 5 scraper workers
for i in range(5):
    t = Thread(target=scraper_worker, args=(q, ))
    t.start()
q.join()
```

**Important!** The `q.join` on the last line is very important. It causes the program to wait for the queue to be emptied before exiting.

### Results

Comparing results is mind blowing. The linear (n00b) scraping method takes over an hour to scrape 5000 pages, and the threaded + queues method takes *less than three minutes*. *

So by adding a few more lines of code to that "quick fix" script, you could save hours of execution time. 

I <3 Python!

<sub>* I ran the tests on my local machine, but not posting the actual results because threading relies greatly on the CPU and that differs for each environment.</sub>

----

### External References

#### Threading

[PyMOTW threading](http://pymotw.com/2/threading/)

[Python Threading and Queues - and why its awesome](http://lonelycode.com/2011/02/04/python-threading-and-queues-and-why-its-awesome/)

#### Queues

[PyMOTW queues](http://pymotw.com/2/Queue/)

