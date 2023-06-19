---
layout: post
title:  "Python Logging"
date:   2021-02-21 03:59:00 +0800
category: Python
categories: [Python]
tags: [Python, Logging]
---
### Who is this post for ?

Software developers / machine learning engineers / data scientists looking to implement logging in their python scripts. The idea of this post is to simply share how I went about implementing logging to meet my requirements, in the hope that it may help someone else who is also starting out in this field.

### Tools this post will be using : 

1. Python 3
3. MacOS (Catalina)

### Let's start :

Logging module comes with Python's standard library. And from what I have understood so far, it is fairly easy to implement when trying to accomplish basic logging tasks, but it gets tricky when one need's some advanced functionality (at least for a novice like me).

**Let me briefly describe my use case** - I had to implement logging in keeping with certain cloud logging standards since the product was to be deployed on cloud. I had to define the logging configurations in a separate file so that if any logging functionality needed modification, it could be changed directly from this fairly simple configuration file, without having to access the actual python script. The logs were required to rotate by size as well as time. The log level and the output path of the generated logs had to be set using environment variables. Also, which logs files would be generated had to be based on this set log level. 

Now let's handle these requirements one at a time, beginning with logging configuration.

Taking directly from the Python documentation, this is how one can configure logging - 

*[`*Configuring Logging`](https://docs.python.org/3/howto/logging.html#configuring-logging)*

*Programmers can configure logging in three ways:*

1. *Creating loggers, handlers, and formatters explicitly using Python code that calls the configuration methods listed above.*
2. *Creating a logging config file and reading it using the [`fileConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.fileConfig) function.*
3. *Creating a dictionary of configuration information and passing it to the [`dictConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig) function.*

Since we need to define configurations in a separate file, points 2 and 3 are relevant for us. If you dig in further, you'll find in the [`Pyhton documentation`](https://docs.python.org/3/library/logging.config.html#configuration-file-format) that - 

*The [`fileConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.fileConfig) API is older than the [`dictConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig) API and does not provide functionality to cover certain aspects of logging. For example, you cannot configure [`Filter`](https://docs.python.org/3/library/logging.html#logging.Filter) objects, which provide for filtering of messages beyond simple integer levels, using [`fileConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.fileConfig). If you need to have instances of [`Filter`](https://docs.python.org/3/library/logging.html#logging.Filter) in your logging configuration, you will need to use [`dictConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig). Note that future enhancements to configuration functionality will be added to [`dictConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig), so it’s worth considering transitioning to this newer API when it’s convenient to do so.*

We will need to implement filters, and a custom handler as well (one which rolls over the log files by size as well as time). Hence it makes sense for us to proceed with [`dictConfig()`](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig).

We can define a dictConfig either directly in a .py file, or a YAML file. I chose to go ahead with YAML since it is more readable compared to the .py format, especially for someone not necessarily familiar with Python, who might be required to modify the configurations file in future.

So let's look at a basic YAML configuration file for logging.

```yaml
version: 1

formatters:
    basicFormatter:
        format: "%(asctime)s - %(name)s - %(levelname)s %(message)s"
        datefmt: "%b %d %H:%M:%S"

handlers:
    consoleHandler:
        class: logging.StreamHandler
        level: DEBUG
        formatter: basicFormatter
        stream: ext://sys.stdout

    fileHandler:
        class: logging.handlers.FileHandler
        level: INFO
        formatter: basicFormatter
        filename: ./info.log

root:
    level: DEBUG
    handlers: [consoleHandler]

loggers:
   basicLogger:
       level: INFO
       handlers: [fileHandler]
       propogate: yes
```

Here we have the basics in place. The 'level' that we have defined in our config file for loggers and handlers helps us ensure that a record below the declared level is discarded. Which means that our 'filehandler' (with 'level' set to 'INFO') will emit a log of level 'INFO' or higher ('WARNING', 'ERROR', 'CRITICAL'), but will not emit a log of level DEBUG. 

But what do you do if you need a filehandler to emit logs of ONLY a particular level, neither higher nor lower levels? This is where filters come in. You can use filters to define the rules based on which a handler should emit a log. 

So let's add filters. To do this we can implement a subclass of the [`logging.Filter`](https://docs.python.org/3/library/logging.html#filter-objects) class. We need our filter method to return 'True' only when the level of the log record is the same as the level passed to our filter object. 

```python
import logging
import yaml

# Filter for handlers
class MyFilter(logging.Filter):
    def __init__(self, level):
        self.__level = level

    def filter(self, logRecord):
        return logRecord.levelname == self.__level
```
Now we can use this 'MyFilter' object in our config file. For this, we first need to add the 'filters' key to our config file and define the filter that we wish to use. To enable the logging system to access the 'MyFilter' object, [special key '()'](https://docs.python.org/3/library/logging.config.html#user-defined-objects) is used, which provides the absolute import path to a callable which returns the 'MyFilter' object. We then add this filter to our handler. Note that filters can be added to both handlers and loggers. In my case, since I was using the same logger to log to different files, I decided to add filters to my file handlers. 

```yaml
disable_existing_loggers: true

formatters:
    basicFormatter:
        format: "%(asctime)s - %(name)s - %(levelname)s %(message)s"
        datefmt: "%b %d %H:%M:%S"

filters:
  myFilterInfo:
    (): __main__.MyFilter
    'level': 'INFO'

handlers:
    consoleHandler:
        class: logging.StreamHandler
        level: DEBUG
        formatter: basicFormatter
        stream: ext://sys.stdout

    fileHandler:
        class: logging.handlers.FileHandler
        level: INFO
        filters: [myFilterInfo]
        formatter: basicFormatter
        filename: ./info.log

root:
    level: DEBUG
    handlers: [consoleHandler]

loggers:
   basicLogger:
       level: INFO
       handlers: [fileHandler]
       propogate: yes
```

Great! Now let's handle the requirement of setting log level and the output path of log files using environment variables. This can be done with the help of [YAML constructors](https://matthewpburruss.com/post/yaml/#constructors-and-representers), which would allow us to take a YAML node and output a Python object. First, lets add the code to construct objects for log filename and log level.

```python
# Constructor to fetch log file path from env vars
def constructor_logFilename(loader: yaml.SafeLoader, node):
    value = loader.construct_scalar(node)
    path, file = value.split('/')
    if os.environ.get(path) is not None and os.path.exists(os.environ.get(path)) and os.path.isdir(os.environ.get(path)):
        return os.path.join(os.environ.get(path), file)
    else:
        return os.path.join('./log', file)

# Constructor to fetch log level from env vars
def constructor_logLevel(loader: yaml.SafeLoader, node):
    value = loader.construct_scalar(node)
    if os.environ.get(value) is not None and (os.environ.get(value) in ['DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL']):
        return os.environ.get(value)
    else:
        return 'ERROR'
 
```

Now, let's [add these constructors](https://pyyaml.docsforge.com/master/api/yaml/add_constructor/) to YAML.

```python
yaml.add_constructor(u'!logFilename', constructor_logfilename)
yaml.add_constructor(u'!logLevel', constructor_logLevel)
```

Now we can use the tags specified above (!logFilename and !logLevel) in our configurations YAML file. Also, if you remember the use case, we need to generate log files based on the log level set using environment variables. So if we set the log level as INFO, we want to generate log files for level INFO and above and not for the DEBUG logs. To do this, I decided to use the root logger, specify the level there, and add multiple file handlers under the root logger for all the log files we wish to generate. This way, we won't have to create multiple loggers. 

Our updated config file looks like this - 

```yaml
disable_existing_loggers: true

formatters:
    basicFormatter:
        format: "%(asctime)s - %(name)s - %(levelname)s %(message)s"
        datefmt: "%b %d %H:%M:%S"

filters:
  myFilterInfo:
    (): __main__.MyFilter
    'level': 'INFO'

handlers:
    consoleHandler:
        class: logging.StreamHandler
        level: DEBUG
        formatter: basicFormatter
        stream: ext://sys.stdout

    fileHandler:
        class: logging.handlers.FileHandler
        level: INFO
        filters: [myFilterInfo]
        formatter: basicFormatter
        filename: !logfilename ENV_VAR_LOG_PATH/my_info.log

root:
    level: !logLevel ENV_VAR_LOG_LEVEL
    handlers: [consoleHandler, fileHandler]

```

We can add multiple file handlers for different log levels that we wish to generate a file for, and then add those handlers to the root logger.

Now let's handle the requirement of log rotation by size as well as time. Out of the box, the Python logging module provides a handler for rotating a file based on time ([TimedRotatingFileHandler](https://docs.python.org/3/library/logging.handlers.html#timedrotatingfilehandler)) and a separate handler for rotating a file based on size ([RotatingFileHandler](https://docs.python.org/3/library/logging.handlers.html#rotatingfilehandler)). 

But we need both functionalities, and so we'll have to create a custom file handler ([reference](https://stackoverflow.com/questions/6167587/the-logging-handlers-how-to-rollover-after-time-or-maxbytes)).

```python
# File handler that rotates by size as well as time
# and also compresses and saves rotated files under folders of the 
# respective month in which these files were created

from logging.handlers import RotatingFileHandler
from logging.handlers import TimedRotatingFileHandler
import gzip
from datetime import datetime, timedelta
import time
import itertools

class EnhancedFileHandler(logging.handlers.TimedRotatingFileHandler, logging.handlers.RotatingFileHandler):
    def __init__(self, filename, mode='a', maxBytes=0, backupCount=0, encoding=None, delay=0, 
                 when='MIDNIGHT', interval=1, utc=False):
        logging.handlers.TimedRotatingFileHandler.__init__(self, filename=filename,
                                                           when=when,interval=interval,
                                                           backupCount=backupCount, 
                                                           encoding=encoding, delay=delay, 
                                                           utc=utc)

        logging.handlers.RotatingFileHandler.__init__(self, filename=filename, mode=mode, 
                                                      maxBytes=maxBytes,
                                                      backupCount=backupCount,
                                                      encoding=encoding, delay=delay)

    isTimeBasedRollover = False

    def computeRollover(self, current_time):
        return logging.handlers.TimedRotatingFileHandler.computeRollover(self, current_time)

    def get_time(self, current_time):
        return current_time - timedelta(seconds=self.interval)

    def doRollover(self):
        # get from logging.handlers.TimedRotatingFileHandler.doRollover()
        # a datetime stamp is appended to the filename when the rollover happens. 
        # The file should be named for the start of the interval, not the time at which rollover occurs. 
        current_time = int(time.time())
        if self.isTimeBasedRollover:
            file_suffix = self.get_time(datetime.today())
        else:
            file_suffix = datetime.today()
        dst_now = time.localtime(current_time)[-1]
        new_rollover_at = self.computeRollover(current_time)

        while new_rollover_at <= current_time:
            new_rollover_at = new_rollover_at + self.interval

        # If DST changes and midnight or weekly rollover, adjust for this.
        if (self.when == 'MIDNIGHT' or self.when.startswith('W')) and not self.utc:
            dst_at_rollover = time.localtime(new_rollover_at)[-1]
            if dst_now != dst_at_rollover:
                if not dst_now:
                  # DST kicks in before next rollover, so we need to deduct an hour
                    addend = -3600
                else:
                  # DST bows out before next rollover, so we need to add an hour
                    addend = 3600
                new_rollover_at += addend
        self.rolloverAt = new_rollover_at

        if self.stream:
            self.stream.close()
            self.stream = None

        path, file_name = os.path.split(self.baseFilename)
        self.dir_log = os.path.abspath(os.path.join(path, str(file_suffix.strftime('%Y-%m'))))
        self.filename = file_name
        if not os.path.isdir(self.dir_log):
            os.mkdir(self.dir_log)

        for i in itertools.count(1):
            nextName = '{}-{}-{}.log.gz'.format(
            os.path.join(self.dir_log, self.filename[:-4]),str(file_suffix.strftime('%Y-%m-															%d')),i)
            if not os.path.exists(nextName):
                with open(self.baseFilename, 'rb') as original_log:
                    with gzip.open(nextName, 'wb') as gzipped_log:
                        shutil.copyfileobj(original_log, gzipped_log)
                os.remove(self.baseFilename)
                self.rotate(self.baseFilename, nextName)
                break

        if not self.delay:
            self.stream = self._open()

    def shouldRollover(self, record):
        if logging.handlers.TimedRotatingFileHandler.shouldRollover(self,record)==1:
            self.isTimeBasedRollover = True
            return 1
        elif logging.handlers.RotatingFileHandler.shouldRollover(self, record)==1:
            self.isTimeBasedRollover = False
            return 1
        else:
            return 0
```

Now we replace the file handler class in our YAML config file with this EnhancedFileHandler class that we've defined. This is what the final config file looks like -

```yaml
disable_existing_loggers: true

formatters:
    basicFormatter:
        format: "%(asctime)s - %(name)s - %(levelname)s %(message)s"
        datefmt: "%b %d %H:%M:%S"

filters:
  myFilterInfo:
    (): __main__.MyFilter
    'level': 'INFO'

handlers:
    consoleHandler:
        class: logging.StreamHandler
        level: DEBUG
        formatter: basicFormatter
        stream: ext://sys.stdout

    fileHandler:
        (): __main__.EnhancedFileHandler
        level: INFO
        filters: [myFilterInfo]
        formatter: basicFormatter
        filename: !logfilename ENV_VAR_LOG_PATH/my_info.log
        maxBytes: 15485760 # 15MB
        when: 'MIDNIGHT'
        interval: 1
        utc: 0
        encoding: utf8

root:
    level: !logLevel ENV_VAR_LOG_LEVEL
    handlers: [consoleHandler, fileHandler]
```

Finally, we load this config file and create our logger!

```python
with open(log_config_path, 'r') as fd:
    config = yaml.load(fd.read(), Loader=yaml.Loader)
logging.config.dictConfig(config)
my_logger = logging.getLogger()
```

**I hope this post was helpful.**

**Cheers!**

