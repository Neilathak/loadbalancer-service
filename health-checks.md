---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-12"

keywords: health check, http, tcp, ports

subcollection: loadbalancer-service

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}

# Performing Health Checks with IBM Cloud Load Balancer
{: #performing-health-checks-with-ibm-cloud-load-balancer}

The health check definitions are mandatory for each of the back-end application ports. The port and protocol under health check configuration must match with defined back-end port and protocol, otherwise the configuration is rejected.

The load balancer conducts periodic health checks to monitor the health of the back-end ports and forwards client traffic to them accordingly. If a given back-end server port is found unhealthy, then no new connections are forwarded to it. The load balancer continues to monitor health of these unhealthy ports, and resumes their use if they are found healthy again by successfully passing two consecutive health check attempts.

The health checks against HTTP and TCP ports are conducted as follows:

* **HTTP:** An `HTTP GET` request against a pre-specified URL is sent to the back-end server port. The server port is marked healthy upon receiving a `200 OK` response. The default `GET` URL is “/” via the GUI, and it can be customized.

* **TCP:** The Load Balancer attempts to open a TCP connection with the back-end server on a specified TCP port. The server port is marked healthy if the connection attempt is successful, and the connection is then closed.

	The default health check interval is 5 seconds, the default timeout against a health check request is 2 seconds, and the default number of retry attempts is 2.
  {: note}
