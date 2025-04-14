                          Introducing Prometheus with Grafana: Metrics Collection and Monitoring
                          
Introduction
Metrics collection and Monitoring system is very important for a healthy and scalable software system. Its your eyes and ears for what is happening inside the system. For an enterprise application this is an absolute requirement not only to know how well the system is performing but also, use this data for all sorts of analysis to measure latencies, find potential bottlenecks and alarming when something bad happens.

I came across Prometheus while working on Kubernetes and Docker (before that I had experience with ELK kind of stack or CloudWatch kind of cloud based services). Once, I got introduced to Prometheus and how it works, for me the question of how to collect metrics properly shifted forever. And it happened for good, which in this post we will explore. I have tried my best to keep things simple, to the point and understandable without any pre-requisite. At the same time, this also means, I would cover mostly basics to get you up and running as fast as possible without going into advanced features. It will also feature a full demo of a Prometheus server setup in a Docker environment with some metrics visualised in Grafana.

“Once I got introduced to Prometheus and how it works, for me the question of how to collect metrics properly shifted forever.”

What is Prometheus
The story of Prometheus is very intriguing, so let me take a min to tell you how it came about. It was originally developed at SoundCloud, by ex-Googlers. But its story goes beyond that, Google had a dynamic container based system for all their systems in production called Borg and was using an in house monitoring solution called Borgmon for the same. But in SoundCloud there were no alternative for the same, other solutions at the time doesn’t fit the need of the dynamic environments (similar to Kubernetes). So, the inception of Prometheus began building an open source solution on the same key pricinples for the similar kind of environment. I think the Author’s sentiment captures it the best,

“They are twins separated at birth.” Kubernetes directly builds on Google’s decade-long experience with their own cluster scheduling system, Borg. Prometheus’s bonds to Google are way looser but it draws a lot of inspiration from Borgmon, the internal monitoring system Google came up with at about the same time as Borg. In a very sloppy comparison, you could say that Kubernetes is Borg for mere mortals, while Prometheus is Borgmon for mere mortals. Both are “second systems” trying to iterate on the good parts while avoiding the mistakes and dead ends of their ancestors.

https://youtu.be/4Pr-z8-r1eo
So what is it basically, Prometheus is an open-source systems monitoring and alerting toolkit (Yes its very moduler!). It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

It got adopted by Cloud Native Computing Foundation as the second project to be adopted right after Kubernetes itself. Today, Prometheus is the de facto monitoring tool used alongside Kubernetes everywhere.

But, I would like to think about it as a time series database with a powerful query language. At least the main Prometheus server is just that. Well that was a mouthful, so let me break it down bit by bit, and also for this post, let’s consider Prometheus to be a DB rather than a metrics collection/alarming system. You will know why it makes more sense in short while. But first, let’s learn about time series DB, what are they good for and how do they help.

Let’s talk about time series Databases
A time series database (TSDB) is a database optimized for time-stamped or time series data. Time series data are simply measurements or events that are tracked, monitored, downsampled, and aggregated over time. This could be server metrics, application performance monitoring, network data, sensor data, events, clicks, trades in a market, and many other types of analytics data. (Internally it uses 64bit floating point XOR based compression algorithm that Prometheus uses under the hood)

A time series database is built specifically for handling metrics and events or measurements that are time-stamped. A TSDB is optimized for measuring change over time. Properties that make time series data very different than other data workloads are data lifecycle management, summarization, and large range scans of many records.

If you look up most popular TSDBs right now, you will find, Prometheus to be in one of the top ones.

Why is a time series database important now?
Time series databases are not new, but the first-generation time series databases were primarily focused on looking at financial data, the volatility of stock trading, and systems built to solve trading. But its scope is no longer limited to just that. The fundamental conditions of computing have changed dramatically over the last decade. Everything has become compartmentalized. Monolithic mainframes have vanished, replaced by serverless servers, micro serverices, and containers. With these kind of architecture, finding the sequence of things happening in real time over many many systems running in parallel requires a monitoring system build on top of solid TSDB foundation.

Push vs Pull Model for Metrics Collection
Metrics are one of the “go-to” standards for any monitoring system of which there are a variety of different types. At its core, a metric is essentially a measurement of a property of a portion of an application or system. Metrics make an observation by keeping track of the state of an object. These observations are some value or a series of related values combined with a timestamp that describes the observations, the output of which is commonly called time-series data.

The canonical example for metrics gathering is website hits, where we regularly collect various data points like the number of times someone visits a particular page and the location or source of the visitor for a given site. Examples of different types of metrics are as follows:

Counters
Timers
Events
Logs
A single metric in and of itself is often not that useful, but when visualized over time especially in conjunction with some mathematical transformation, metrics can give great insight into a system and its behaviour. Following are some common transformations for metrics:

Sum
Average
Median
Percentiles
Standard deviation
Rates of change
In Push-based monitoring the application being monitoring becomes an emitter of data “pushing” metrics and events on some time interval or a constraint violation. This approach typically favors a decentralized structure and has the advantage of not requiring the monitoring system to “pre-register” the monitored component as the emitter pushes data to the configured destination upon start. But it suffers from several drawbacks and loses its clear visibility if a push based agent dies or doesn’t respond over time. Also, the amount of data pushed needs be optimized in this approach(because it is a part of the application and application throughput in some sense.)

Pull/polling based solutions works differently in that they pull data which are exposed via different services and endpoints. are quite common in monitoring and historically favors a centralized organizational structure, although this is not a requirement. Primarily, in pull-based monitoring, the monitoring system asks or queries a monitored component, for example, pinging a host or to see if a website is up.

Prometheus is a pull based system which pulls data from configured sources at regular intervals.

Architecture
Most Prometheus components are written in Go, making them easy to build and deploy as static binaries for all types of target/systems. Prometheus scrapes metrics from instrumented jobs, either directly or via an intermediary push gateway for short-lived jobs. It stores all scraped samples locally and runs rules over this data to either aggregate and record new time series from existing data or generate alerts

Prometheus’s main features are:

a multi-dimensional data model with time series data identified by metric name and key/value pairs
PromQL, a flexible query language to leverage this dimensionality
Time series collection happens via a pull model over HTTP
pushing time series is supported via an intermediary gateway
targets are discovered via service discovery or static configuration
multiple modes of graphing and dashboarding support
The Prometheus ecosystem consists of multiple components, many of which are optional:

the main Prometheus server which scrapes and stores time series data
client libraries for instrumenting application code
a push gateway for supporting short-lived jobs
special-purpose exporters for services like HAProxy, StatsD, Graphite, etc. which sends metrics from an existing system or software.
an alertmanager to handle alerts

![image](https://github.com/user-attachments/assets/77fb75d1-40f4-41fc-8ce4-852043bb0cae)

When does it fit?
Prometheus works well for recording any purely numeric time series. It fits both machine-centric monitoring as well as monitoring of highly dynamic service-oriented architectures like Kubernetes. In a world of microservices, its support for multi-dimensional data collection and querying is a particular strength.

Prometheus is designed for reliability, to be the system you go to during an outage to allow you to quickly diagnose problems. Each Prometheus server is standalone, not depending on network storage or other remote services. You can rely on it when other parts of your infrastructure are broken, and you do not need to setup extensive infrastructure to use it.

When does it not fit?
Prometheus values reliability. You can always view what statistics are available about your system, even under failure conditions. If you need 100% accuracy, such as for per-request billing, Prometheus is not a good choice as the collected data will likely not be detailed and complete enough. In such a case you would be best off using some other system to collect and analyze the data for billing, and Prometheus for the rest of your monitoring.

Prometheus Setup
For the demonstration purposes we will use Docker to make things really easy and reproducible anywhere. Here is a simple Dockerfile which sets up the stage for us.

It starts off with Ubuntu 18.04 (bionic) official image
Installs some of the tools we will be using like wget , screen and vim
It downloads the latest binary releases for Prometheus, node_exporter and alertmanager (As we said before, Prometheus is highly modular)
It also downloads Grafana which will be used later for Visualization
Exposes the default ports from the respective services

FROM ubuntu:bionic
MAINTAINER Arindam Paul, arindampaul1989@gmail.com

RUN  apt-get update \
  && apt-get install -y wget \
  && apt-get install -y screen \
  && apt-get install -y vim

WORKDIR /root
RUN wget -nv https://github.com/prometheus/prometheus/releases/download/v2.18.0/prometheus-2.18.0.linux-amd64.tar.gz
RUN wget -nv https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
RUN wget -nv https://github.com/prometheus/alertmanager/releases/download/v0.20.0/alertmanager-0.20.0.linux-amd64.tar.gz
RUN wget -nv https://dl.grafana.com/oss/release/grafana-6.7.3.linux-amd64.tar.gz

# node_exporter
EXPOSE 9100

# prometheus
EXPOSE 9090

# grafana
EXPOSE 3000

# alertmanage


Now, we will create a simple docker compose file to prepare our docker image to run, only thing it does is, it gives us a handy name to work with which will expose multiple ports to the host for us. With this we can access them from host machine. I generally prefer docker compose a lot than writing long docker run commands with options.
![image](https://github.com/user-attachments/assets/6b00d4bc-daaf-47d3-a48d-4649fbeef3eb)



Once these two files are there, you can go to the folder containing them and run the service from docker-compose in interactive mode. Note that --service-ports is very important option, it allows us to do the port binding right (which is disabled by default).
![image](https://github.com/user-attachments/assets/cd4f095f-3e4e-4509-83c2-3d84b5ec2840)

![image](https://github.com/user-attachments/assets/a6180007-c2b4-4fe8-ae95-fbd29322533c)



Once you run the command, you will see bunch of outputs corresponding to the build steps in Dockerfile. You will also see it downloading the required binaries and tools as follows,
![image](https://github.com/user-attachments/assets/0b41e89c-4343-408c-8491-176a972d979d)


Finally, it will drop you in an interactive prompt inside the running container prometheus_demo_run.
![image](https://github.com/user-attachments/assets/fa29e40d-bbe4-469f-a0ef-aca7627cf7ff)


If you want to see the port mappings of the container to the host, we can go to another terminal keeping this one on (else the running instance will exit because it has nothing to do), and run docker-compose ps you will see the status of you container. Right now it’s in a bash shell and the state is up and the ports are exposed correctly as shown below,


Port Bindings with Host
![image](https://github.com/user-attachments/assets/71c31e19-7581-479b-ad12-923bb2ce0810)

This is all the setup needed, now the fun time begins. If you list the directory you will all our required tools already downloaded for us.

We will first start Prometheus server to get things rolling, you can follow along by simple extracting the package.

![image](https://github.com/user-attachments/assets/3ae7d4d9-98f8-4950-a751-c84faf362140)

And running prometheus command inside. It starts up the server with some default configuration. Here you can see it starts off with 15 days of default retention period.

![image](https://github.com/user-attachments/assets/2bf72a5d-1ec9-4dab-8db1-89e51986f536)

Also, if you notice in the end, it actually starts the Database with message “TSDB started” which indicates the time series DB is at the heart of it. Lastly, it reads a file on the same directory called prometheus.yml which holds the configuration for the server. We will look at it more closely in next sections.
![image](https://github.com/user-attachments/assets/0418bd97-d175-4b98-b847-ea15dce84cdf)


Now, in your host machine (laptop) if you open up the URL http://localhost:9090/ You will get something like this.
![image](https://github.com/user-attachments/assets/90c9fa6e-dcfe-4456-846f-2e74f8198d9c)


This indicates your Prometheus server is up and running successfully, ready to be used. Before we get into any details, I would like to show you the configuration with which Prometheus was loaded in the first step.

![image](https://github.com/user-attachments/assets/cd303bd6-0cc8-45e9-80f3-16154c4a24e8)


You notice its a very simple configuration like interval of polling (scrape interval), where to poll from etc. Interesting thing to note here is that, Prometheus itself exposes its metrics on the same port in /metrics path for the Prometheus Server to pull data about its own health. Let’s see what it is, it you open up http://localhost:9090/metrics sure enough you will simple key/values with some comments. This is a simple text based format used by Prometheus for more on this you can refer to the documentation.

![image](https://github.com/user-attachments/assets/8b5cd000-8134-4f65-8b17-5ed9608ef9f6)

But essentially it means, we already have metrics that we are collecting and can be queried from Prometheus Server, so let’s get back to the server. If you click on the available metrics dropdown, sure enough all those metrics are there.

![image](https://github.com/user-attachments/assets/fe43d632-ca4a-4a6b-a086-1a04efeb9964)

Let’s select a metric to explore little more, let’s say we want to know how much memory is allocated for Prometheus server, we select go_memstats_alloc_bytes (Just an example, you can use whatever you want).

If I execute this query you will see, I get one value. By default, any metric query gives the last recorded result.

![image](https://github.com/user-attachments/assets/92e5836f-76e1-4657-b06b-048ad0ea7c31)

But, if we want to know how this has changed over time, we can use the PromQL query language to do so as shown in the example, its simple and expressive language in fact this is as powerful as the Time Series DB itself and Prometheus is nothing but these two together. More on this.

![image](https://github.com/user-attachments/assets/8f68889a-7fa4-4394-bd7e-bcb430242bad)

Now, we get all the values recorded in last 5 minutes and you can see all of them have a value and a timestamp associated with it. You can also visualize the data in the visualize tab. With this you can clearly see, how the memory consumption is happening for Prometheus server.
![image](https://github.com/user-attachments/assets/4fd78850-6a5c-4959-b20d-4f7306a8375f)


Exporters in Prometheus
There are a number of libraries and servers which help in exporting existing metrics from third-party systems as Prometheus metrics. This is useful for cases where it is not feasible to instrument a given system with Prometheus metrics directly (for example, MySQL Server, HAProxy or Linux system stats).

Prometheus has huge ecosystem of exporters, almost any popular software that you want to use it. We will look at one of official ones in details here,

Node Exporter (Machine Health)
Node exporter is a simple health metric exporter for you system. If collects many many system level metrics by looking at the /proc files of your system. We have already downloaded node exporter in our setup step, to understand more let’s fire it up in our Docker shell (new shell using screen)
![image](https://github.com/user-attachments/assets/8d1f5ea4-b5d0-42bb-a1e1-10d64e31b664)


You can see once we start the node exporter, it starts the collectors for many many individual stats and exposes them to be consumed at the port 9100.
![image](https://github.com/user-attachments/assets/ba0dad8b-1b12-4705-927a-1dd9a524ce05)

![image](https://github.com/user-attachments/assets/66c1899f-b381-4bb2-832d-482baad76b5f)


If you go to the URL http://locahost:9100/metrics you will see a very similar kind of metrics page we saw earlier with Prometheus server metrics.

![image](https://github.com/user-attachments/assets/34dbd0da-f18f-45f0-a0d3-7b76ecd3538c)

This time as you can see if has the keys start with node_ which differentiates the key space.

Now the next step is how do it collect these metrics in Prometheus server. For that, we will stop our Prometheus server (I am using screen to open multiple shells in my session) and edit our config file like this,

![image](https://github.com/user-attachments/assets/f8e08bb1-c74f-4e22-8232-368ea37f0dcf)

We added one more target for our server, now the server will poll both of these targets at the specified time interval to collect metrics.

With this we start the Prometheus server and verify the config is loaded successfully,
![image](https://github.com/user-attachments/assets/955319bf-c962-4393-94c0-cbd0352ed102)


You can see now in the targets section we have two targets as expected. In next sections we will explore more on Queries and Visualisation based on this node exporter data only.

Prometheus Queries
The whole reason we opted for a Time Series DB like Prometheus is so that we can query the data in a very flexible way. And Prometheus lives upto this promise with it’s powerful PromQL language. Let’s take a look at a simple metric first,
![image](https://github.com/user-attachments/assets/7e031e17-cc6c-4d40-aaf0-c4f7743dc61a)


You can see with node exporter, it collects nested info as to which is the host and instance sending this data.

One important thing to note here is that if you add more nodes and export data using node exporter, the metrics will have the same keys but will differ in the nested context like above where the instance or cpu will be different. So in Prometheus metrics are collected by service and not my host. Very important to keep that in mind.

We can add more conditions to the query for drilling down to a specific metric like this, it gives us the latest recorded value for the metric,
![image](https://github.com/user-attachments/assets/8f67a455-6860-42bb-af86-631869c6abfd)


We can also visualize the same data in Graph tab which shows the data collected over time,
![image](https://github.com/user-attachments/assets/f6529a0b-dd4e-4411-ae5b-a7e039f7e90e)


If you want to see the actual data points, we can add [5m] time frame to our query and execute the same, here are the values for the last 5m interval shown in the diagram below,
![image](https://github.com/user-attachments/assets/39c326dd-9717-4870-a970-1274a15aa35e)


So, with this simple setup you can see how much you can get for free without doing pretty much no work. Prometheus also supports publishing Custom metrics as well in a very powerful way, in all possible languages. Huge community again thrives on the aspect of choices and flexibility to mix and match your solutions.

Grafana
Now that we have a Prometheus server collecting metrics for us, we would want a very powerful visualisation tool which can used for all sorts of dashboard and monitoring requirements.

Grafana is a great tool in this regard. It integrates with almost all popular sources of metrics server like Prometheus and allows us much more visual tooling which ultimately converts them to complex queries to Prometheus server, but hey! we don’t have to worry about them.

So, let’s extract the downloaded Grafana in a new shell (use Screen to one a new shell Crtl+A+C) and start the grafana-server as shown below,
![image](https://github.com/user-attachments/assets/88a0600f-ac29-477c-94f1-545122907cb7)

It will start the Grafana server on default port 3000.
![image](https://github.com/user-attachments/assets/c847d1b9-9f93-4876-93b8-9b8ae3845b96)


If we go to http://localhost:3000 we will get to the grafana homepage. We can login to grafana with default username admin and password admin. (For first time users it might ask you change the password intially, feel free to set it up they way you like it)
![image](https://github.com/user-attachments/assets/be6f1120-34dc-42b4-8f18-c1ec1bd920ff)


Once logged in you will see, first thing Grafana needs is some data source to work with. In our case, we will go ahead and click on the add data source link.
![image](https://github.com/user-attachments/assets/1f56180e-d248-4dbf-bee4-6da867fdf63c)


In the options you can see Grafana is capable of talking to multiple Databases, Prometheus being on the top :)
![image](https://github.com/user-attachments/assets/e69c25f9-0eeb-4acf-a865-f2302abd411a)


So we select Prometheus and and provide the URL of our Prometheus Server.
![image](https://github.com/user-attachments/assets/ee30ad37-0ffd-43c8-8e0d-ced6a4d62fd7)

We can save and test the configuration to make sure the data source is working properly.
![image](https://github.com/user-attachments/assets/4c710805-0d53-43b6-9a5f-3d62762c1daf)

With that setup out of the way, we can now use Grafana to create dashboards and Alarms (yes Alarms can be done in Grafana layer itself, though we will show you alertmanager which works in tandem with Prometheus)
![image](https://github.com/user-attachments/assets/933e3c1c-e997-42f0-93b4-95799163ef1a)

For fun, let’s adda Query for memory usage of the whole node. Grafana has very powerful autocomplete feature, just to illustrate, here is how you get autocomplete in the UI,

![image](https://github.com/user-attachments/assets/1b314a29-3499-448b-b3ca-b06344313ee9)

So, if we start typing, node_ we will get a bunch of auto completes, for now, we will use the metric node_memory_Active_bytes to see the memory usage.
![image](https://github.com/user-attachments/assets/bba7c6d4-6dbe-4ea8-bd3c-7220fd8db8b0)

Immediately you can see data starts to flow in in a nice graph. Give yourself a round of applause if everything worked out so far.

But, this is not the end as we said, with Grafana we can add Alarms right where graphs are configured. Let’s do one for fun, so we go to add alarms,

![image](https://github.com/user-attachments/assets/41b2354d-37ed-49f3-bd37-0addee5d9caa)

And configure the alarm conditions, thresholds, you can see the alarm indicator comes alive showing you exactly where the alarm is getting set to.
![image](https://github.com/user-attachments/assets/f958d515-4323-485b-8b19-ce790305e8da)

What we learnt is just tip of the iceberg on what you can do with Grafana. For example, in Grafana dashboard can be exported and imported in JSON format (Completely sharable). And again, there is a huge community who has already shared a lot of dashboard which we can reuse to learn and tweak to our needs.

In this case, for this simple setup, if we want to see the full visualation in comprehensive dashboard, I have used Node Exporter Full community dashboard.

And viola!! This is what we get from it by default, Amazing isn’t it :)
![image](https://github.com/user-attachments/assets/105b2f82-d9c2-4c81-a68c-ddb10ce85b47)


Conclusion
So, in this post we learned several things, including what Prometheus is and how it works, how to set it up, how we can get metrics from exporters and query them. How we can visualize the same in Grafana.

I wanted to continue with Metric Format, Custom Metric Collection in Prometheus from Application code (whitebox monitoring) and using alert manager to trigger alerts right from Prometheus layer. But looking at the length of this post, I am keeping it just focused on the basics. I will be writing another follow up post on the remaining concepts.

Stay Tuned!





##  How to Run the Project 

1. **Install Docker Desktop on Windows**  
   Download from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

2. **Clone this repo**

```bash
git clone https://github.com/rudranis/devops_task.git
cd devops_task
