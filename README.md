𝐏𝐫𝐨𝐣𝐞𝐜𝐭 𝐓𝐢𝐭𝐥𝐞: 𝐈𝐧𝐭𝐫𝐨𝐝𝐮𝐜𝐢𝐧𝐠 𝐏𝐫𝐨𝐦𝐞𝐭𝐡𝐞𝐮𝐬 𝐰𝐢𝐭𝐡 𝐆𝐫𝐚𝐟𝐚𝐧𝐚: 𝐌𝐞𝐭𝐫𝐢𝐜𝐬 𝐂𝐨𝐥𝐥𝐞𝐜𝐭𝐢𝐨𝐧 𝐚𝐧𝐝 𝐌𝐨𝐧𝐢𝐭𝐨𝐫𝐢𝐧𝐠

Tools & Tech Stack

Prometheus: For scraping and storing metrics.

Grafana: For visualizing metrics from Prometheus.

Node Exporter (or any other exporters): To collect system metrics.

Step-by-Step Project Setup

1️⃣ Install Prometheus
Option 1: Manual Install

Download Prometheus

Extract and run:

./prometheus --config.file=prometheus.yml

Option 2: Docker

docker run -p 9090:9090 prom/prometheus

Install Node Exporter

To collect system metrics:

docker run -d -p 9100:9100 prom/node-exporter

Or manually:

./node_exporter

Configure Prometheus

In prometheus.yml, add the Node Exporter job:

scrape_configs:

  - job_name: 'node_exporter'
  - 
    static_configs:
    
      - targets: ['localhost:9100']

Then restart Prometheus.

Install Grafana

Option 1: Manual Install

Download Grafana

Start Grafana: ./bin/grafana-server

Option 2: Docker

docker run -d -p 3000:3000 grafana/grafana

Connect Prometheus to Grafana

Login to Grafana (localhost:3000, default login: admin/admin)

Go to Settings → Data Sources → Add data source

Select Prometheus

Set URL: http://localhost:9090

Click Save & Test

Create Dashboard in Grafana

Go to Create → Dashboard

Add a new panel

Query example:

node_cpu_seconds_total{mode="idle"}

Add more panels:

CPU Usage

Memory Usage

Disk IO

Network Usage


Sample docker-compose.yml

version: '3'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"


Final Dashboard Should Have

💻 System Metrics: CPU, Memory, Disk, Network

📊 Custom Queries & Graphs

🔔 Alerts (if enabled)

🎨 Clean layout with panel grouping


 Introduction to Prometheus & Monitoring Systems
 
Why Monitoring Matters
In scalable systems, monitoring helps you:
Track performance
Detect issues early
Trigger alerts when problems occur

🔍 What is Prometheus?
Open-source monitoring tool inspired by Google’s Borgmon
Adopted by CNCF (like Kubernetes)
Pull-based model: Scrapes metrics from endpoints
Uses PromQL for querying
Stores data in a Time Series Database (TSDB)

📊 Time Series Database (TSDB)
Stores data with timestamps — useful for:
App/server metrics
IoT/sensor data
Market activity
Prometheus uses XOR compression for efficient time-based analysis.

📥 Push vs Pull Metrics

Model	Pros	Cons
Push	Decentralized	Possible data loss
Pull	Centralized, scalable	Requires open endpoints
Prometheus uses Pull — ideal for Kubernetes & dynamic setups.

🏗️ Prometheus Architecture
Prometheus Server – Core engine
Client Libraries – Instrument your apps
Exporters – Get metrics from services like MySQL, HAProxy
Push Gateway – For short-lived jobs
AlertManager – Sends alerts (Slack, email)
Grafana – Dashboards and visualization

✅ When to Use Prometheus
Use it when:
You're working with microservices or Kubernetes
You need reliable, time-series metrics
Avoid if:

You need 100% accurate billing
Logging high-frequency events

🐳 Docker Setup (Minimal)
dockerfile
Copy
Edit
# Base image
FROM ubuntu:bionic

# Install required tools
RUN apt-get update && \
    apt-get install -y wget

# Set working directory
WORKDIR /root

# Download Prometheus ecosystem tools
RUN wget -nv https://github.com/prometheus/prometheus/releases/download/v2.18.0/prometheus-2.18.0.linux-amd64.tar.gz && \
    wget -nv https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz && \
    wget -nv https://github.com/prometheus/alertmanager/releases/download/v0.20.0/alertmanager-0.20.0.linux-amd64.tar.gz && \
    wget -nv https://dl.grafana.com/oss/release/grafana-6.7.3.linux-amd64.tar.gz

# Expose ports
EXPOSE 9100 9090 3000

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
