# Project-Base-Learning-7
DEVOPS TOOLING WEBSITE SOLUTION

## DEVOPS TOOLING WEBSITE SOLUTION

## PROJECT TASK
- Implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

## BACKGROUND KNOWLEDGE
- In previous [Project 6](https://www.darey.io/docs/project-6-step-1/) I implemented a WordPress based solution that was filled with content and can be used as a full fledged website or blog. Moving further I added some more value to my solutions that my DevOps team could utilize. I want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.

- The tools I want our team to be able to use are well known and widely used by multiple DevOps teams, so I will introduce a single DevOps Tooling Solution that will consist of:

- 1 [Jenkins](https://www.jenkins.io) - free and open source automation server used to build [CI/CD](https://en.wikipedia.org/wiki/CI/CD) pipelines.

- 2 [Kubernetes](https://kubernetes.io) – an open-source container-orchestration system for automating computer.

- 3 [Jfrog Artifactory](https://jfrog.com/artifactory/) – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.

- 4 [Rancher](https://www.rancher.com/products/rancher) – an open source software platform that enables organizations to run and manage [Docker](https://en.wikipedia.org/wiki/Docker_(software)) and Kubernetes in production.

- 5 [Grafana](https://grafana.com) – a multi-platform open source analytics and interactive visualization web application.

- 6 [Prometheus](https://prometheus.io) – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

- 7 [Kibana](https://www.elastic.co/kibana/) – Kibana is a free and open user interface that lets you visualize your Elasticsearch [Elasticsearch](https://www.elastic.co/elasticsearch/) data and navigate the [Elastic](https://www.elastic.co/elastic-stack/) Stack.

Note: Do not feel overwhelmed by all the tools and technologies listed above, we will gradually get ourselves familiar with them in upcoming projects!

Side Self Study : Read about Network-attached storage (NAS), Storage Area Network (SAN) and related protocols like NFS, (s)FTP, SMB, iSCSI. Explore what Block-level storage is and how it is used by Cloud Service providers, know the difference from Object storage.
On the example of AWS services understand the difference between Block Storage, Object Storage and Network File System.

## Setup and technologies used in Project 7

- As a member of a DevOps team, I implemented a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

- In this project I implemented a solution that consists of following components:

- Infrastructure: AWS
- Webserver Linux: Red Hat Enterprise Linux 8
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 8 + NFS Server
- Programming Language: PHP
- Code Repository: [GitHub](https://github.com/darey-io/tooling)

- On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using [Network File Sytem (NFS)](https://en.wikipedia.org/wiki/Network_File_System) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.
- 
![7_1](https://github.com/EzeOnoky/Project-Base-Learning-7/assets/122687798/0d10cd6d-5ecf-4cbc-a713-095719a85375)

It is important to know what storage solution is suitable for what use cases, for this – you need to answer following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this you will be able to choose the right storage system for your solution.

