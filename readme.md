Skyline
=======

[![Build Status](https://travis-ci.org/etsy/skyline.png)](https://travis-ci.org/etsy/skyline)

Introduction
------------

![x](https://raw.github.com/etsy/skyline/master/screenshot.png)

Skyline is a real-time* anomaly detection* system*, built to enable passive
monitoring of hundreds of thousands of metrics, without the need to configure a
model/thresholds for each one, as you might do with Nagios. It is designed to be
used wherever there are a large quantity of high-resolution timeseries which
need constant monitoring. Once a metrics stream is set up (from StatsD or
Graphite or other source), additional metrics are automatically added to Skyline
for analysis. Skyline's easily extendible algorithms automatically detect what
it means for each metric to be anomalous. After Skyline detects an anomalous
metric, it surfaces the entire timeseries to the webapp, where the anomaly can be
viewed and acted upon.

Read the details in the [wiki](https://github.com/etsy/skyline/wiki).

Instalation
-----------

### Python 

Ensure you have all the python libraries and pip, on Debian Wheezy you will also require python-dev.

```bash
sudo apt-get install python-dev python-pip
```

Install known requirements for pip.

`sudo pip install -r requirements.txt` 

Install numpy, scipy, pandas, patsy, statsmodels, msgpack_python in that order.

On Debian Wheezy you will have numpy, scipy and statsmodels in ports and the remainers in pip.

```bash
sudo apt-get install python-numpy python-scipy python-scikits.statsmodels
sudo pip install patsy msgpack_python 
```

You may have trouble with SciPy. If you're on a Mac, try:

```bash 
sudo port install gcc48
sudo ln -s /opt/local/bin/gfortran-mp-4.8 /opt/local/bin/gfortran
sudo pip install scipy
```

On Centos, yum should do the trick. If not, hit the Googles, yo.

### Configuration

Copy example file for settings, this is where you'll add the Graphite host and more.

`cp src/settings.py.example src/settings.py`

If using Vagrant you will need to forward the web application as well as update the bound IP.

On Vagrantfile:

```config.vm.network :forwarded_port, guest: 1500, host: 1500```

And on src/settings.py:

```python
WEBAPP_IP = ‘0.0.0.0'
```

Add directories: 

``` 
sudo mkdir /var/log/skyline
sudo mkdir /var/run/skyline
sudo mkdir /var/log/redis
```

### Redis 

Download and install the latest Redis release.

On Debian Wheezy you will have the current Redis version available on the backport. After updating the source.list and updating you will be able to install 2.6.

On /etc/apt/sources.list:

```deb http://{SOURCE}.debian.org/debian/ wheezy-backports main```

Replacing {SOURCE} with your current configuration URL.

```bash
sudo apt-get update
sudo apt-get -t wheezy-backports install redis-server
```

When Redis is installed it might start running with the default config, don't forget to kill it before starting it with the Skyline configuration.

```sudo pkill redis-server```

### Start 'er up

```bash
cd skyline/bin
sudo redis-server redis.conf
sudo ./horizon.d start
sudo ./analyzer.d start
sudo ./webapp.d start
```

By default, the webapp is served on port 1500.

Check the log files to ensure things are running.

Extras
-------

### Gotchas

* If you already have a Redis instance running, it's recommended to kill it and
restart using the configuration settings provided in bin/redis.conf

* Be sure to create the log directories.

### Hey! Nothing's happening!
Of course not. You've got no data! For a quick and easy test of what you've 
got, run this:
```
cd utils
python seed_data.py
```
This will ensure that the Horizon
service is properly set up and can receive data. For real data, you have some 
options - see [wiki](https://github.com/etsy/skyline/wiki/Getting-Data-Into-Skyline)

Once you get real data flowing through your system, the Analyzer will be able
start analyzing for anomalies!

### Alerts
Skyline can alert you via email! In your settings.py, add any alerts you want
to the ALERTS list, according to the schema `(metric keyword, recipient,
expiration seconds)`. For every anomalous metric, Skyline will search for the
given keyword and alert the proper recipient. To prevent alert fatigue, Skyline
will only alert once every <expiration seconds> for any given metric.

### How do you actually detect anomalies?
An ensemble of algorithms vote. Majority rules. Batteries __kind of__ included.
See [wiki](https://github.com/etsy/skyline/wiki/Analyzer)

### Architecture
See the rest of the
[wiki](https://github.com/etsy/skyline/wiki)

### Contributions
We actively welcome contributions. If you don't know where to start, try
checking out the [issue list](https://github.com/etsy/skyline/issues) and
fixing up the place. Or, you can add an algorithm - a goal of this project
is to have a very robust set of algorithms to choose from.

Also, feel free to join the 
[skyline-dev](https://groups.google.com/forum/#!forum/skyline-dev) mailing list
for support and discussions of new features.

(*depending on your data throughput, *you might need to write your own
algorithms to handle your exact data, *it runs on one box)
