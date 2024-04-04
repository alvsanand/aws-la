# AWS-LA (AWS Log Analyzer)

## Description

This script allows you to easily import various AWS log types into an Elasticsearch cluster running locally on your computer in a docker container.

## Supported AWS Log Types

- ELB access logs
- ALB access logs
- VPC flow logs
- [Route53 query logs][r53-query-logs]
- Apache access logs

## Future Log Types Supported

- Cloudtrail audit logs
- Cloudfront access logs
- S3 access logs
- Others to come!

## Steps Automated

The script configures everything that is needed in the ELK stack:

- Elasticsearch:

  - indices
  - mappings
  - ingest pipelines

- Kibana:

  - index-patterns
  - field formatting for index-pattern fields
  - importing dashboards, visualizations, and dashboards
  - custom link directly to the newly created dashboard

## Installation Steps

- Install Docker, [Docker for Windows][docker-for-windows] or [Docker for Mac][docker-for-mac]
- Install python3
- Clone this git repository:

  `git clone https://github.com/mike-mosher/aws-la.git && cd aws-la`

- Install requirements (Virtualenv):

  ``` bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r ./requirements.txt
  ```

## Running the Script

- Fix Elasticsearch error (Linux):

  `sudo sysctl -w vm.max_map_count=262144`

- Fix Elasticsearch error (MacOS): https://docs.rancherdesktop.io/how-to-guides/increasing-open-file-limit/

- Bring the docker environment up:

  `docker-compose -p elk_cluster_la_vpclogs up -d`

- Verify that the containers are running:

  `docker ps`

- Verify that Elasticsearch is running:

  `bash -c 'while [[ "$(curl -s -o /dev/null -w ''%{http_code}'' localhost:9200/_cluster/health?pretty)" != "200" ]]; do sleep 5; done'`

- Browse to the link provided in the output by using `cmd + double-click`, or browse directly to the default Kibana page:

  `http://localhost:5601`

- Valid log types are specified by running the `--help` argument. Currently, the valid logtypes are the following:

  ``` text
  elb                 # ELB access logs
  alb                 # ALB access logs
  vpc                 # VPC flow logs
  r53                 # Route53 query logs
  apache              # apache access log ('access_log')
  apache_archives     # apache access logs (gunzip compressed with logrotate)
  ```

- You can import multiple log types in the same ELK cluster. Just run the command again with the new log type and log directory:

  ``` bash
   python2.7 importLogs.py --logtype vpc --logdir ~/logs/vpc-flowlogs/
  ```

- When done, you can shutdown the containers:

  `docker-compose -p elk_cluster_la_vpclogs down -v`

## Screenshots / Examples

- Python output: ![Python script output][cli-output]

  - As you can see, I was able to import 12.5 million VPC flowlogs in around 2 hours

- Searching for traffic initiated by RFC1918 (private) IP addresses:

  - Browse to Discover tab, and enter the following query in the query bar:

  `source_ip_address:"10.0.0.0/8" OR source_ip_address:"172.16.0.0/12" OR source_ip_address:"192.168.0.0/16"`

  ![Search for RFC1918 Traffic][search-rfc1918]

  - Alternately, you can search for all traffic initiated by Public IP addresses in the logs:

  `NOT (source_ip_address:"10.0.0.0/8" OR source_ip_address:"172.16.0.0/12" OR source_ip_address:"192.168.0.0/16")`

  ![Search for non-RFC1918 Traffic][search-non-rfc1918]

  - Search for a specific flow to/from a specific ENI:

  `interface-id:<eni-name> AND (source_port:<port> OR dest_port:<port>)`

  ![Search flow to Specific ENI][search-eni]

  - Note: VPC Flow Logs split a flow into two log entries, so the above search will find both sides of the flow and show packets / bytes for each

- Dashboard imported for VPC Flow Logs: ![VPC Dashboard][vpc-dashboard]

- Dashboard imported for ALB Access Logs: ![ALB Dashboard][alb-dashboard]

[alb-dashboard]: examples_screenshots/ALB_Dashboard_Screenshots/ALB_Dashboard.jpg?raw=true
[cli-output]: examples_screenshots/VFL_example_12.5m_documents_imported.png?raw=true
[docker-for-mac]: https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac
[docker-for-windows]: https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows
[r53-query-logs]: https://aws.amazon.com/about-aws/whats-new/2017/09/amazon-route-53-announces-support-for-dns-query-logging/
[search-eni]: examples_screenshots/VPC_Dashboard_Screenshots/Search_for_both_sides_of_a_flow_record_for_a_specific_ENI.png?raw=true
[search-non-rfc1918]: examples_screenshots/VPC_Dashboard_Screenshots/Search_for_non_RFC1918_traffic.png?raw=true
[search-rfc1918]: examples_screenshots/VPC_Dashboard_Screenshots/Search_for_RFC1918_traffic.png?raw=true
[vpc-dashboard]: examples_screenshots/VPC_Dashboard_Screenshots/VPC_Flow_Logs_Dashboard.jpg?raw=true
