> **As we were writing our response to the RFI for a DTIC Master Data Repository (ref: https://www.fbo.gov/index?id=6954cb16269968c2c87cab3223dcbb40), I realized that much of the information I was writing are topics of discussion when talking to peers, clients, etc.**
>
>And since I **hate** writing white papers, I realized that it may make sense to simply publish our RFI response (sans sensitive info of course) on Github.
>
>Of course, this response is in context of the RFI - so some approaches are in alignment with the requirements - but I think overall it is a decent approach when trying to move fast with indexing data and moving towards an API centric architecture.
>
>Enjoy!

> [@jobrieniii](http://twitter.com/jobrieniii)

-


#Master Data Repository Technology and Implementation Support RFI - HQ000242730294000 <small>DECEMBER 30, 2014</small>

## <img src='https://pbs.twimg.com/profile_images/431746805458411520/WuQ2kqy6.png' width=35> Company Info
```

{
  "name": "540.CO LLC",
  "www": "http://540.co",
  "github": "http://github.com/540co"
  "duns": "078812845",
  "cage": "6W6J5",
  "street": "3930 Rolling Hills Drive",
  "city": "Cumming",
  "state": "GA",
  "zip": "30041",
  "point_of_contact": {
    "name": "John O'Brien III",
    "phone": "4077180732",
    "email": "jobrieniii@540.co",
    "github": "http://github.com/jobrieniii"
  },
  "procurement_info": {
    "sam_api_url": "https://api.data.gov/sam/v1/registrations/0788128450000?api_key=DEMO_KEY",
    "business_size": "small",
    "naics": [511210,518210,541330,541330,541511,541512,541513,541519,541611,541990,611420],
    "gsa_mobis_status": "540 is not currently on this vehicle, but have partners who are"
  }
}

```

##<img src='https://s3.amazonaws.com/540-font-awesome/png/list30.png' width=30> Our Approach

Many database technologies have emerged on the market to handle the **ingestion**, **storage**, **indexing** and **exposure** in alignment with DTIC's goal to deploy a **Master Data Repository (MDR)** to _"manage, control, search, analyze and disseminate S&T data."_

However, we have not seen one tool in the market that alone could meet every requirement outlined in the RFI without the addition of plugins / extensions, since many of requirements jump between different data types (XML, JSON, DOC files) and various integration models (SQL, API, SPARQL).  

As a result it will be important that we deploy an **open source solution** with an _active development / support community_ that can be quickly iterated upon as data sources and integration requirements evolve and data storage / analysis techniques evolve.

Based upon our experience with numerous NoSQL technologies both in R&D and Production environments, **we are recommending that DTIC implement Elasticsearch as the foundation for the MDR**.

**[Elasticsearch (ES)](http://www.elasticsearch.org/) is an active open source / vendor supported document based NoSQL data store that has seen rapid adoption in the Enterprise and Government space.** 

The product is **highly scalable** and will meet the following key RFI requirements:

- **Index and store data** with diverse schemas leveraging the schema-less nature of ES

- [**REST APIs**](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/index.html) for all interaction swith ES data store to kickstart application development and provide dynamically generated API endpoints as data `types` are inserted into `indexes`

	**Example ES API endpoints**
	
	`https://api.dtic.mil/reports/tems/` where `index = report` and `type = tems`
	
	`https://api.dtic.mil/budget/p40/` where `index = budget` and `type = p40`	

- [**Search**](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl.html) including full text, faceted, structure / unstructured queries across large datasets, scoring / relevancy, deep pagination, scroll / scan (similar to cursors)

- [**Aggregation queries**](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations.html) to support analytics, metrics, counts, etc.

- **[Javascript library](http://www.elasticsearch.org/guide/en/elasticsearch/client/javascript-api/current/)** to support front end app development

- [**Kibana**](http://www.elasticsearch.org/guide/en/kibana/current/using-kibana-for-the-first-time.html) plugin to provide out of the box search, visualization and analytics

- Numerous data [**harvesting / river plugins**](http://www.elasticsearch.org/?s=rivers) (organic and 3rd party)

##<img src='https://s3.amazonaws.com/540-font-awesome/png/book95.png' width=35>Our Playbook

###1. Establish foundation <small>SOW REFERENCE A, B, K, L</small>

Based upon the requirements as outlined in the RFI we would first
- Deploy (2) 3-node [**Elasticsearch (ES) clusters**](http://www.elasticsearch.org/) (one on the NIPR and one on SIPR) to ensure accessibility and load balancing across nodes
- Install the [**Marvel**](http://www.elasticsearch.org/overview/marvel) plugin to provide an easy way for administrators to monitor the clusters
- Install a reverse proxy (such as [nginx](http://nginx.org/)), or leverage a proxy already implemented within DTIC, to establish an initial security barrier in front of the ES APIs

> **Want to implement in the cloud? <small>SOW REFERENCE K</small>**
>
> If part of the implementation of the MDR will also include leaning forward with a cloud infrastructure, we recommend leveraging [Amazon Web Service (AWS) Govcloud](http://aws.amazon.com/govcloud-us/), which was [recently granted a provisional ATO](http://aws.amazon.com/compliance/dod-csm-faqs/) for data impact levels 3-5 (which would handle the NIPR side of the MDR).
> 
> As an AWS partner, 540 has hands-on experience with AWS Govcloud and currently operates numerous CAC-enabled forward facing and administrative tools in that environment.

> We also have experience with leveraging DISA's Milcloud if that is more in line with the DAA/AO's "move to the cloud."

-

###2. Extend foundation <small>SOW REFERENCE A,B,D</small>

To meet some of the pre-defined requirements outlined in the RFI, the following ES plugins will then be implemented:

- [**elasticsearch-mapper-attachments**](https://github.com/elasticsearch/elasticsearch-mapper-attachments) to support the requirements to index [documents / files](http://tika.apache.org/1.5/formats.html)

- [**eea/eea.elasticsearch.river.rdf**](https://github.com/eea/eea.elasticsearch.river.rdf) to support the harvesting of SPARQL endpoints

- [**logstash**](http://www.elasticsearch.org/overview/logstash/) to support the harvesting of timestamped logs (such as web access logs)

In addition (and depending on the timing of the implementation) a strategy would be defined for **securing** indexed data / documents either leveraging the 
- [**elasticsearch-security-plugin**](https://github.com/salyh/elasticsearch-security-plugin)
- or the soon to be released [**ES Shield plugin**](http://www.elasticsearch.org/overview/shield)

>We are guessing the formal release of Shield will be around the time of the [Elasticsearch Conference] (http://www.elasticsearch.org/blog/its-on-announcing-our-first-user-conference-elasticon15/) - March 2015

-

###3. Document APIs / Establish Visualization <small>SOW REFERENCE: C,E</small>

To document the APIs for "indexing" and "searching" for data, [**Swagger API docs**](http://swagger.io) will be installed to document key ES API resource endpoints.  

**This will provide us a way to document API resources and provide interactive documentation for internal and partner use.**

In addition, ES's [**Kibana**](http://www.elasticsearch.org/guide/en/kibana/current/using-kibana-for-the-first-time.html) will be installed to provide an initial set of visualization / search tools that could be used in production and for prototyping apps that require a more refined UX.

-

###4. Index, expose, monitor and operate <small>SOW REFERENCE: C,D,E,F,G,H,I</small>

At this point, we will begin the iterative process of:

- Indexing data
- Documenting index / type specific ES API resources to "ingest", "search for" and "get" data
- Defining and deploying the the best front end app / UX for the given data resource
- Establish scripts / tools for monitoring quality of data as it is indexed
- Establish operational processes, scripts, tools to maintain the MDR integrity and availability 

When **ingesting data** we will need to prioritize which data will be moved (leveraging the table shown on page 8/9/10 of the Draft SOW) and determine if the data will be continuously indexed or migrated in a single shot. 

**This will further refine the type of "connector" that is implemented and/or how it is used.**

> For example, if the data will still be **mastered in the current source system going forward**, we will want to establish a [river](http://www.elasticsearch.org/guide/en/elasticsearch/rivers/current/) between the source and MDR to allow for continual flow of data.  

> However, if the data will be **migrated once and a new front end app will be used to access / maintain the data**, we will most likely attack the problem differently.

We will also work with the DTIC team to ensure the **"right" UX is defined and deployed for a given data resource** for a given data resource while tackling questions such as:

- How will the user search and/or use the data (keyword, semantic, etc)?

- Does the user need the ability to "bulk" download / export data and in what file formats (XML, CSV, PDF, etc)?

- How will the data be managed (in cases where the MDR is also the master data source and acts as a transactional data store via the APIs)?

- What type of questions / analytics will be asked of the data?
 
> This will result in the build out of a collection of **common and unique apps / tools** that use **common data resources** as indexed and exposed via by the MDR and are lightweight and easy to maintain by leveraging **HTML 5 / Javascript** and **frameworks such as [Bootstrap](http://getbootstrap.com/) and [AngularJS](https://angularjs.org/)**

##<img src='https://s3.amazonaws.com/540-font-awesome/png/cogs3.png' width=35> Rough Architectural Drawing

>You can download a high resolution version at:

>[**https://s3.amazonaws.com/dtic-rfi-12302014/dtic\_mdr\_rfi\_v1_12292014.png**](https://s3.amazonaws.com/dtic-rfi-12302014/dtic_mdr_rfi_v1_12292014.png)

<img src='https://s3.amazonaws.com/dtic-rfi-12302014/dtic_mdr_rfi_v1_12292014.png'/>

##<img src='https://s3.amazonaws.com/fedapi_io/global/fedapi_logo_black_100x100.png' width=32> FedAPI - We are doing something very similar :-)

As a company R&D project we are in the process of rolling out [**FedAPI**](http://fedapi.com), a **data indexing** and **API platform** designed to **accelerate analytics, common operating pictures, data sharing and application development on public government data sources**.

This service leverages many of the approaches and similar technologies (including a **clustered Elasticsearch implementation, Amazon Web Services, Swagger, etc.**) outlined above behind the scenes to harvest and expose the data resources to citizens, corporations and agencies via easy to use APIs.

>**As an example relevant to DTIC**, here is a link to the **DoD Budget data**  as indexed on FedAPI after being harvested from the numerous XML files that are used to prepare the PDFs that are submitted to Congress. 

> [**https://fedapi.io/www/overview/dodbudget**](https://fedapi.io/www/overview/dodbudget)

> **This index was recently used by the [Office of Performance Assessments and Root Cause Analysis (PARCA)](http://www.acq.osd.mil/parca/) to support an analysis of RDTE R3 Exhibits for leadership.**

<img src='https://s3.amazonaws.com/dtic-rfi-12302014/fedapi_dod_budget_catalog.png'/>

##<img src='https://s3.amazonaws.com/540-font-awesome/png/group41.png' width=35> Our Experience

**540** is a technology consulting firm, focused on helping Federal / DoD clients innovate like start-ups by leveraging the technologies and design patterns that drive leading Internet technology companies.

We offer solutions and services including:

- TECH STRATEGY – Tech play books designed for your mission and your optempo (with a push to go faster) that leverage cutting edge tech design patterns 

- SOFTWARE DEVELOPMENT – Web apps, mobile apps, wearables, systems integration, security / information assurance

- API DESIGN / DEV – Data sharing and transactional web services to enable and empower your internal developers, partners, and public consumers

- DATA ANALYTICS – Innovative data analytics to collect, correlate, share and visualize important information at the right time

- DEV / OPS  – Operations designed to scale your platforms, secure you systems, monitor / scale your platforms and wow your customers

In an effort to keep up with the accelerating pace of technology evolution, especially in the area of software design and data analytics, 540 remains active in the consumer and enterprise technology start-up arena.  **We invest heavily in R&D and the support of open source / open data discovery, aggregators and analytics projects, including platforms such as [FedAPI](https://fedapi.com/), a platform designed to collect, correlate and expose public government data to app developers and analysts.**

We are also **[Amazon Web Services (AWS) / GovCloud partner](http://www.aws-partner-directory.com/PartnerDirectory/PartnerDetail?Name=540.co+LLC)** helping our clients leverage the recently ATO'ed AWS GovCloud environment.

**Department of Defense (DoD) clients include [Defense Procurement and Acquisition Policy (DPAP)](http://www.acq.osd.mil/dpap/) and [AT&L ARA](http://www.acq.osd.mil/ara/about-ara.htm).**


## <img src='https://s3.amazonaws.com/540-font-awesome/png/usd.png' width=35> ROM

###Labor

[REMOVED]

-

###Vendor Support

[REMOVED]


## <img src='https://s3.amazonaws.com/540-font-awesome/png/warning18.png' width=35> Challenges

**Unfortunately, no one single data store is going to meet every need perfectly without sub-optimizing something.**

As a result some of the capabilities outlined in the RFI collide with the approach document based NoSQL data stores like Elasticsearch and others have taken, such as ACID transaction integrity, support of query languages (such as SQL and support of business intelligence tools that typically like having an RDBMS warehouse (though rivers are starting to emerge to better integrate with NoSQL data stores and export strategies are relatively easy to deploy depending on the BI tool).

While additional plugins / rivers could be leveraged and/or built to support the various capability needs, we would highly recommend continuing to evaluate / refine the requirements as new data is ingested / exposed and applications / integrating partners are migrated over to leverage the core capabilities of the MDR and APIs.

In cases where a specific capability is a MUST HAVE, we often recommend using the right tool for the right job and establishing a synchronization / river strategy that keeps the stores in synch.






