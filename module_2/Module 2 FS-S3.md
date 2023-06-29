# Module 2: Federated Search for Amazon S3

In the last module you configured AWS VPC Flow Logs to be sent to an Amazon S3 bucket for long-term storage.  In this module, you will configure Splunk Federated Search for Amazon S3 (FS-S3) to search AWS VPC Flow Log residing in an S3 bucket _without_ ingesting the data into Splunk.  FS-S3 works by leveraging [Amazon S3](https://aws.amazon.com/s3/) to store data, and [AWS Glue](https://aws.amazon.com/glue/) to create a data schema for Splunk Cloud to search against.  FS-S3 is great for infrequently searching across large volumes of data.

Specifically for this module and for Forthly, after configuring FS-S3 in this module you will be:
1. Demonstrating to auditors that AWS VPC Flow Log data is being kept in S3 for long retention times
2. Retrieve data from S3 that has aged out of Splunk to help the SOC with a security investigation

You'll be searching against a a different Amazon S3 bucket than what you configured in the last module.  This new S3 bucket has more data to search against what Splunk has placed there via ingest actions over the past few minutes!

FS-S3 has a few parts and there are a few different steps to configure it that involve both the Splunk Admin and AWS Admin.  FS-S3 uses Splunk and AWS-based technologies like Amazon S3, and AWS Glue to search the data in the S3 buckets.  The full documentation and configuration instructions will be available once FS-S3 is released but the configuration overview is:

1. Save data in an Amazon S3 bucket that can be searched.  The data needs to be in a format Amazon Glue can read, and a full list of that is available in the AWS documentation here: ([https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-format.html](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-format.html)).
2. Configure an AWS Glue database ([https://docs.aws.amazon.com/glue/latest/dg/define-database.html](https://docs.aws.amazon.com/glue/latest/dg/define-database.html)) and AWS Glue table ([https://docs.aws.amazon.com/glue/latest/dg/tables-described.html](https://docs.aws.amazon.com/glue/latest/dg/tables-described.html)) to store the data schema.
3. Use an AWS Glue crawler ([https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html](https://docs.aws.amazon.com/glue/latest/dg/add-crawler.html)) to scan over the files in the Amazon S3 bucket to determine the file schema, and store it in the Glue database and table.
4. Configure a Federated Search Provider in Splunk to search the data in the Amazon S3 bucket.

All of these AWS-side components have already been configured for you in this module, so you'll only need to handle that fourth part of configuring FS-S3 in Splunk.  A CloudFormation template containing the resource definitions and configurations used to configure the AWS-side components can be found [here](https://github.com/preeves-splunk/pla1750b/blob/v1/cloudformation-per-attendee.yaml).

It’s worth noting here that in this workshop you're only searching data sent to an Amazon S3 bucket via ingest actions.  However, that’s only due to workshop constraints.  There’s a wide variety of data formats Amazon Glue can read ([https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-format.html](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-etl-format.html)) that are not related to Splunk.  If you have data already in an Amazon S3 bucket that can be read by Amazon Glue, it can be searched by FS-S3 as well regardless of what generated the data.

More documentation will be available when Federated Search for Amazon S3 officially launches!

## Task 1: Configure the Amazon S3 Federated Search Provider and Federated Index in Splunk Cloud

In this task you'll be configuring Splunk Cloud to search VPC Flow Log data residing in an Amazon S3 bucket.  Normally this requires coordination between the AWS Admin and Splunk Admin, but everything AWS-side (including the Glue catalog, Glue Crawler, Glue resource policy and S3 bucket policies) are already created and configured for you.

1. In the Splunk Cloud environment, navigate to **Settings** > **Federated Search**.

![Splunk Cloud with settings display shown with Settings and Federated Search highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_1.png?raw=true)

2. Start the process of adding a new federated search provide by clicking the **Add federated provider** button, selecting **Amazon S3** and clicking **Next**.

![Splunk Cloud add federated provider pane shown with Add Federated Search Provider, the Amazon S3 option, and Next button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_2.png?raw=true)

3. On the `Add Amazon S3 Provider` pane, fill out the following fields:
	- **Federated provider name:** `pla1750b_workshop`
	- **AWS Account ID:** `139817522827`
	- **Glue Data Catalog database**: This needs to start with the Splunk Cloud name but replace the `-` (dashes) with `_` (underscores), and ending in `_glue_database`.  See the image below for an example.
	- **Glue Data Catalog tables**: `pla1750b_public_fs_s3_bucket`
	- **Amazon S3 locations**: `s3://pla1750b-public-fs-s3-bucket/`

![Splunk Cloud Add Amazon S3 Provider pane shown with required input fields highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_3.png?raw=true)

4. Finish creating the Amazon S3 Federated Search Provider by clicking **Generate Policy**, then clicking both check mark boxes next to **Warning and consent** and **Confirmation that Requestor Pay is turned off**, then clicking **Create provider**.

![Splunk Cloud with Add Amazon S3 Provider pane shown with checkmark boxes and Create provider button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_4.png?raw=true)

5. Now that the Federated Search has been created, you need to create a Federated Index.  To start that, click the **Create Federated Index** button.

![Splunk Cloud with settings display shown with Create federated index button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_5.png?raw=true)

6. On the `Create a Federated Index` pane, fill out the following fields:
	- **Federated Index Name**: `vpcFlowLogs`
	- **Dataset Name**: `pla1750b_public_fs_s3_bucket`
	- **Time Field**: `time`
	- **Time Format**: `%UT`
	- **Unix time field**: `_time`

![Splunk Cloud with Create a Federated Index pane shown with required fields highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_6.png?raw=true)

7. You'll need to add 3 partition time settings.  You can set those by clicking the **Add new field** button three times, then setting the following fields:
	- **Level 1**:
		- **Time Field**: `year`
		- **Time format**: `%y`
		- **Type**: `string`
	- **Level 2**:
		- **Time Field**: `month`
		- **Time format**: `%m`
		- **Type**: `string`
	- **Level 3**:
		- **Time Field**: `day`
		- **Time format**: `%d`
		- **Type**: `string`
8. Click **Save**

![Splunk Cloud create a Federated Index pane shown with the partition time settings, Add new field button, and Save button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_7.png?raw=true)

9. To test this configuration, run the search below for **all time** and make sure results are returned with no errors:

```
| sdselect * from federated:vpcflowlogs
```

Please note that it may take several minutes for the proper AWS-based permissions to be deployed to Splunk so that it can search the data without error.  If you do encounter an error, please wait a few minutes then try running the search again.

![Splunk Cloud with results from search | sdselect * from federated:vpcflowlogs with search highlighted in red and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/1_8.png?raw=true)

## Task 2: Search Data Using FS-S3 to Show Long-Term Retention

Now that your FS-S3 provider has been created, you can search data in the Amazon S3 bucket without ingesting it.  The Forthly compliance team is working with auditors to show that you're storing VPC Flow Logs long-term.  They've determined that a sample of logs from mid-June as well as a count of events per AWS Account will satisfy the auditors questions.

The new `sdselect` command is used to query data stored in Amazon S3 made searchable through FS-S3.  It uses a SQL-like syntax structure and the full documentation for this will be released when FS-S3 comes out.  For the workshop though, you will incrementally build the query in each of the following steps.  For each one of these steps read the description for what the query is trying to accomplish then run the query.

1. Make sure the base search runs:

```
| sdselect * from federated:vpcflowlogs
```

![Splunk Cloud with results from search | sdselect * from federated:vpcflowlogs with search highlighted in red and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/2_1.png?raw=true)

2. Modify the time range picker to be June 13th 2023 by clicking **All time**, then setting the **Date Range** to be between **6/13/2023 00:00:00** and **6/13/2023 24:00:00**, then re-run the query.  

![Splunk Cloud with results from search | sdselect * from federated:vpcflowlogs shown and time range picker being changed with search and time range highlighted in red and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/2_2.png?raw=true)

In the real world, you would export these results and send them to the compliance team.

3. The compliance team would also like to know how many AWS accounts are sending events.  Fortunately in the data for this module, the AWS account number is set to be the source, and the `sdselect` command can do stats-like SQL functions.  Run the query below  to calculate the number of events per source, and leave the time range picker for the query set to June 13 2023:

```
| sdselect count from federated:vpcflowlogs groupby source
```

![Splunk Cloud with results from search| sdselect count from federated:vpcflowlogs groupby source shown with search highlighted in red and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/2_3.png?raw=true)


## Task 3: Search Amazon S3 for Security Investigation

The Forthly SOC would like help investigating an ongoing security incident.  Specifically, they'd like to know every IP a specific ENI communicated with between June 17th and June 23rd.  The ENI in question is `eni-0212b9abe7101dbcb` and it’s in the `958172947816` AWS account.

Just like the previous task, for each one of these steps read the description for what the search is trying to accomplish then run the query.

1. Make sure the base query runs over the timeframe of **June 17th through June 23rd**:

```
| sdselect * from federated:vpcflowlogs
```

![Splunk Cloud with results from search| sdselect * from federated:vpcflowlogs groupby source shown over June 17th through June 23rd timeframe with the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/3_1.png?raw=true)

2. Query for only the time and event fields:

```
| sdselect time, event from federated:vpcflowlogs
```

Notice how only the selected `time` and `event` fields are being returned.

![Splunk Cloud with results from search | sdselect time, event from federated:vpcflowlogs groupby source shown over June 17th through June 23rd timeframe with the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/3_2.png?raw=true)

3. Query for just events in the AWS Account ID `958172947816`:

```
| sdselect time, event from federated:vpcflowlogs where source=958172947816
```

Notice how only events involving the AWS account number `958172947816` are shown.

![Splunk Cloud with results from search | sdselect time, event from federated:vpcflowlogs where source=958172947816 shown over June 17th through June 23rd timeframe with the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/3_3.png?raw=true)

4. Add a filter to just look for events from `eni-0212b9abe7101dbcb`:

```
| sdselect time, event from federated:vpcflowlogs where source=958172947816 AND event like "%eni-0212b9abe7101dbcb%"
```

Notice how only events with the string `eni-0212b9abe7101dbcb` are returned. 

![Splunk Cloud with results from search| sdselect time, event from federated:vpcflowlogs where source=958172947816 AND event like "%eni-0212b9abe7101dbcb%" shown over June 17th through June 23rd timeframe ith the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/3_4.png?raw=true)

5. Increase the `LIMIT` to make sure all of the events are being retrieved from the Amazon S3 bucket

```
| sdselect time, event from federated:vpcflowlogs where source=958172947816 AND event like "%eni-0212b9abe7101dbcb%" LIMIT 100000000
```

Notice how there are more than the default 100,000 events being returned.

![Splunk Cloud with results from search| sdselect time, event from federated:vpcflowlogs where source=958172947816 AND event like "%eni-0212b9abe7101dbcb%" LIMIT 100000000 over June 17th through June 23rd timeframe ith the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/3_5.png?raw=true)

6. Data retrieved via the `sdselect` command doesn't know about sourcetypes or search-time field extractions, other SPL commands like `rex` need to be used in order to extract fields at search time.  Use the query below to extract the relevant CIM fields for VPC Flow Logs  (the regular expression was taken from the Splunk Add-on for Amazon Web Services):

```
| sdselect time, event from federated:vpcflowlogs where source=958172947816 AND event like "%eni-0212b9abe7101dbcb%" LIMIT 100000000
| rex field=event "^\s*(\d{4}-\d{2}-\d{2}.\d{2}:\d{2}:\d{2}[.\d\w]*)?\s*^(?<version>2{1})\s+(?<account_id>[^\s]{7,12})\s+(?<interface_id>[^\s]+)\s+(?<src_ip>[^\s]+)\s+(?<dest_ip>[^\s]+)\s+(?<src_port>[^\s]+)\s+(?<dest_port>[^\s]+)\s+(?<protocol_code>[^\s]+)\s+(?P<packets>[^\s]+)\s+(?<bytes>[^\s]+)\s+(?<start_time>[^\s]+)\s+(?<end_time>[^\s]+)\s+(?<vpcflow_action>[^\s]+)\s+(?<log_status>[^\s]+)"
```

Notice how CIM field extractions like `dest_port`, and `src_ip` are now in the event table.

![Splunk Cloud with results from search shown ith the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/3_6.png?raw=true)

7. Add a `stats` command to calculate the total number of bytes transmitted between the suspect ENI, each source IP it held, and each destination IP it talked to:

```
| sdselect time, event from federated:vpcflowlogs where source=958172947816 AND event like "%eni-0212b9abe7101dbcb%" LIMIT 100000000
| rex field=event "^\s*(\d{4}-\d{2}-\d{2}.\d{2}:\d{2}:\d{2}[.\d\w]*)?\s*^(?<version>2{1})\s+(?<account_id>[^\s]{7,12})\s+(?<interface_id>[^\s]+)\s+(?<src_ip>[^\s]+)\s+(?<dest_ip>[^\s]+)\s+(?<src_port>[^\s]+)\s+(?<dest_port>[^\s]+)\s+(?<protocol_code>[^\s]+)\s+(?P<packets>[^\s]+)\s+(?<bytes>[^\s]+)\s+(?<start_time>[^\s]+)\s+(?<end_time>[^\s]+)\s+(?<vpcflow_action>[^\s]+)\s+(?<log_status>[^\s]+)"
| stats sum(bytes) as total_bytes by src_ip, dest_ip
| sort - total_bytes
```

![Splunk Cloud with results from search shown](https://github.com/preeves-splunk/pla1750b/blob/main/module_2/3_7.png?raw=true)

Again, in the real world, you would export these results and send them to the SOC for further investigation.