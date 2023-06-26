# Module 1: Ingest Actions

Now that you're acquainted with your new Splunk environment, it's time to get some work done!  You've received a few requests:

1. Management wants to reduce cost by removing non-valuable data
2. The compliance team wants to prevent sensitive information from being ingested into Splunk
3. The cloud storage team needs events re-routed to another index
4. The data science team wants to send data to a Amazon S3, where their data lake is hosted

Fortunately, ingest actions can help with all of these requests!  Ingest actions is the best way to route, filter, and mask data in Splunk, before the data is ingested.  Ingest actions consists of rules that apply to a ruleset.  Rulesets get applied on a per-sourcetype basis.  So this means that any ruleset (or rule in a ruleset) will get applied to a whole sourcetype unless specific events are filtered for or against in the rulesets.

In Splunk Cloud, ingest actions works by modifying `props.conf` and `transforms.conf` via the search head, then making it easy to deploy those rulesets to the indexers.  The indexers apply all the rules defined in ingest actions at index-time; none of the work is done on search heads.  This can have an impact on SVC usage on the indexers, so it’s important to create well-written rules to minimize this impact.  Also, since the work is being done by the indexers at index-time, only index-time fields and metadata fields (like `host`, `index`, `source`, and `sourcetype`) are available to be used in rule filters.

In this module, you will use ingest actions to clean up, reduce, redact, and re-route data in Splunk before it gets ingested.  You'll build out several rulesets, then towards the end of the module check the work.  If you're curious, the full documentation for ingest actions can be found on [docs.splunk.com](https://docs.splunk.com/Documentation/SplunkCloud/latest/Data/DataIngest).

## Task 1: Reduce Data Ingestion Volume by Dropping Non-Value Events

After investigating the Splunk Cloud environment, you've discovered that firewall-related events in the  `firewall` index are the largest data source.  You discussed it with the networking and security teams, and have all come to the conclusion that any events related to ICMP (ping) do not need to be ingested into Splunk.  In this task, you'll be configuring Splunk Cloud to drop those events before they're ingested.

1. Run the search below to identify the ICMP-related events that can be dropped:

```
index=firewall sourcetype=cisco:asa 106023 OR 302020 OR 302021 OR 313005 OR 106015 OR 106006 OR 106001 OR 302022 OR 302023 OR 302024 OR 302025 OR 302026 OR 302027 OR 4001*
```

![Search results from index=firewall sourcetype=cisco:asa 106023 OR 302020 OR 302021 OR 313005 OR 106015 OR 106006 OR 106001 OR 302022 OR 302023 OR 302024 OR 302025 OR 302026 OR 302027 OR 4001* with search highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_1.png?raw=true)

2. Create a new ingest actions ruleset by clicking **Settings** > **Ingest Actions**, then clicking **New Ruleset**.

![Splunk Cloud interface with Settings pane shown with Settings and Ingest Actions text highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_2.png?raw=true)

![Splunk Cloud interface with blank ingest actions page shown with New Rulseset button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_3.png?raw=true)

3. Enter a descriptive **Ruleset Name** to indicate that this ruleset applies to the `cisco:asa` sourcetype events and will be used to reduce data, such as `ciscoasa-reduction`.
4. Configure the ingest actions ruleset to run against the `cisco:asa` sourcetype data by clicking the **Select sourcetype...** box, typing `cisco:asa`, and clicking **Use cisco:asa**.

![Splunk Cloud ingest actions interface with cisco:asa sourcetype being selected with the ruleset name, select sourcetype, and Use cisco:asa highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_4.png?raw=true)

5. Populate the `Data Preview` pane with sample data by clicking the **Sample** button.  As you create and apply rules, you will see the data in the Data Preview section of the page change.

![Splunk Cloud ingest actions interface with cisco:asa sample events with the Sample button highlighted in red, and the Data Preview pane with data highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_5.png?raw=true)

6. Create a new rule by clicking the **+ Add Rule**, and selecting the **Filter using Regular Expressions** rule.

![Splunk Cloud ingest actions interface with Filter using Regular Expressions rule being selected with + Add Rule button and Filter using Regular Expression filter highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_6.png?raw=true)

7. Configure this rule by leaving the **Source Field** set to `_raw` so the regex applies to the `_raw` field, and setting the **Drop Events Matching Regular Expression** field to the regex `(106023|302020|302021|313005|106015|106006|106001|302022|302023|302024|302025|302026|302027|4001\d+)`, then clicking **Apply** to apply the rule to the sample data.

You may need to scroll down the list of sample data in the Data Preview section, but you'll notice events matching the regex are highlighted in red which indicates these events will be dropped.

![Splunk Cloud ingest actions interface with applied via regex with regex , and Apply button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_7.png?raw=true)

Also, notice how additional information around how much data was reduced is provided, including the number of events that were modified.  In the screenshot below, you can see that 8% of the events were dropped because of that regex!  That number might be different in your Splunk Cloud environment.

![Splunk Cloud ingest actions interface with applied via regex with dropped events and percentage chagned highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_8.png?raw=true)

8. View the settings that will be applied to the `props.conf` and `transforms.conf` files by clicking the **Preview Config** button.  This is a good way to check to see how Splunk is going to apply this rule set.  No action is needed here, this step was just to show you how you can preview the configurations before they get deployed.  
9. Click the **Close** button.

![Splunk Cloud ingest actions interface showing preview of props.conf and transforms.conf with Preview Config, Save, and Close buttons highlighted in red and the props.conf and transforms.conf settings highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/1_9.png?raw=true)

10. Save and deploy this ruleset by clicking the **Save** button.   If at any point you receive an error when trying to save these rules, try saving the ruleset again.

## Task 2: Reduce Event Size

After taking another look at the firewall logs in the `firewall` index , you notice that the Cisco eStreamer data can be reduced by removing fields that don't contain valuable data.  After checking with the security and firewall teams, it's agreed these fields in the data can be removed.

1. Search for the Cisco eStreamer data by running the search below:

```
index=firewall sourcetype=cisco:estreamer:data
```

Notice how many of the values in the events consist of non-valuable data such as `null`, `N/A`, and `0`s.

![Splunk Cloud search results index=firewall sourcetype=cisco:estreamer:data with search highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/2_1.png?raw=true)

2. Create a new ingest actions ruleset by clicking **Settings** > **Ingest Actions**, then clicking **New Ruleset**.

![Splunk Cloud with Settings menu shown with Settings and Ingest Actions highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/2_2.png?raw=true)

![Splunk Cloud ingest actions page with the New Ruleset button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/2_3.png?raw=true)

3. Enter a descriptive **Ruleset Name** to indicate that this ruleset applies to the `cisco:estreamer:data` sourcetype events and will be used to reduce data, such as `ciscoestreamerdata-reduction`.
4. Configure the ingest actions ruleset to run against the `cisco:estreamer:data` sourcetype data by clicking the **Select sourcetype...** box, typing `cisco:estreamer:data`, and clicking **Use cisco:estreamer:data**.
5. Click the **Sample** button to retrieve sample data and populate the Data Preview pane.

![Splunk Cloud ingest actions ruleset with cisco:asa sourcetype being selected with the ruleset name, sourcetype selection, and use cisco:estreamer:data text highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/2_4.png?raw=true) 

6. Add a new rule by clicking the **+ Add Rule**, and selecting the **Mask with Regular Expression** option.
7. Configure ingest actions to remove some of the non-valuable fields by setting the following fields in the rule:
	- **Match Regular Expression:** `([a-z\_]+=(""\s))`
	- **Replace Expression:** Don't put anything here, just select the field and tap your space bar to enter a space character.
8. Click the **Apply** button.

Notice how this simple regular expression is only looking for key/value pairs with no data, and  is already reducing event size by 6%.

![Splunk Cloud ingest actions ruleset with regex mask applied with the regular expression, replace expression, Apply button, and + Add Rule button highlighted in red and removed fields and data reduction size highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/2_5.png?raw=true) 

9. To remove more non-valuable fields change the **Match Regular Expression** field value to `([a-z0-9\_]+=(""|(b')?[0-]+(')?|TLS_NULL_WITH_NULL_NULL|Unknown|"Not Checked"|[0:]+|N\/A)\s)` to remove all of the non-valuable fields in the events, then click **Apply**.

Notice how much more of the data is now being removed, with 64% of the data being reduced in the screenshot below!

![Splunk Cloud ingest actions ruleset with regex mask applied with the regular expression and Apply button highlighted in red, and Data Preview pane highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/2_6.png?raw=true) 

10. View the detailed statistics about data reduction by clicking the **Ruler** button on the upper-right side of the `Data Preview` pane.
11. Click the **Save** button to save & deploy the new ruleset.

![Splunk Cloud ingest actions ruleset page with statistics shown with the ruler icon and Save button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/2_7.png?raw=true) 

## Task 3: Mask Sensitive Data in Events

The compliance team has requested that any sensitive data be removed from events before the events are ingested in Splunk Cloud.  Specifically, they have found that credit card numbers that are being ingested in the purchase records and need those credit card numbers need to be removed.

1. Run the search below to view the purchase records:

```
index=purchases sourcetype=web_purchases 
```

Notice how there are credit card numbers in the events.

![Splunk Cloud search results from index=purchases sourcetype=web_purchases with the search highlighted in red and the credit card numbers in the search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/3_1.png?raw=true)

2. Create a new ingest actions ruleset by clicking **Settings** > **Ingest Actions**, then clicking **New Ruleset**.
3. Enter a descriptive **Ruleset Name**, such as `web_purchases-redaction` to indicate that this ruleset applies to the `web_purchases` sourcetype and that this ruleset is intended to redact data.
4. Configure the ingest actions ruleset to run against the `web_purchases` sourcetype data by clicking the **Select sourcetype...** box, typing `web_purchase`, and clicking **Use web_purchase**.
5. Click the **Sample** button to retrieve sample data and populate the Data Preview pane.

![Splunk Cloud ingest actions pane with web_purchase sourcetype selected, with ruleset name, select sourcetype, use web_purchases and Sample button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/3_2.png?raw=true)

5. Create a new rule by clicking the **+ Add Rule**, and selecting the **Filter using Regular Expressions** rule.

![Splunk Cloud ingest actions pane selecting Filter using Regular Expressions rule with the + Add Rule button and Filter using Regular Expression option highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/3_3.png?raw=true)

6. Configure the rule to mask (redact) credit card numbers by filling out the following fields:
	- **Match Regular Expression:** `(credit_card=")(\d*)(\d{4})`
	- **Replace Expression:** `\1XXXX\3`
7. Click the **Apply** button.

![Splunk Cloud ingest actions pane selecting Filter using Regular Expressions rule with regular expressions and Apply button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/3_4.png?raw=true)

8. Save and deploy this rule set by clicking the **Save** button.

## Task 4: Change Destination Index

The cloud storage team has needs all cloud storage-related events sent to the `storage_services` index, but right now those events are being routed to the `main` index.  In this task, you will re-route those events to the correct index.

1. Run the search below to view the cloud storage-related events in the `main` index:

```
index=main sourcetype=o365:management:activity Workload=OneDrive
```

Notice how this search is only looking for OneDrive events.

![Splunk Cloud search results from index=main sourcetype=o365:management:activity Workload=OneDrive with the search highlighted in red and the Workload OneDrive data in the event highlighted in green ](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/4_1.png?raw=true)

2. Create a new ingest actions ruleset by clicking **Settings** > **Ingest Actions**, then clicking **New Ruleset**.
3. Enter a descriptive **Ruleset Name**, such as `o365managementactivity-routing` to indicate that this ruleset applies to the `o365:management:activity` sourcetype and that this ruleset is intended to route data.
4. Configure the rule to use the `o365:management:activity` sourcetype by clicking the **Select sourcetype...** box, typing `o365:management:activity`, and clicking **Use o365:management:activity**.
5. Retrieve sample data and populate the `Data Preview` pane by clicking the **Sample** button.

![Splunk Cloud ingest actions page with o365:managment:activity sourcetpye defined with the rule set name, sourcetype, and Sample button highlighted in red, and sample data highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/4_2.png?raw=true)

6. Add a new rule by clicking the **+ Add Rule** then selecting the **Set Index** rule.

![Splunk Cloud ingest actions with the + Add Rule button and Set Index rule highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/4_3.png?raw=true)

7. Configure the rule to re-route the OneDrive data to the `storage_services` index by setting the following fields:
	- **Condition:** `Regex`
	- **Regular Expression:** `("Workload": "OneDrive")}$`
	- **Set index as:** `storage_services`
8. Apply the rule by clicking the **Apply** button.

Notice how the `index` value in the events has been changed from `main` to `storage_servies`.

![Splunk Cloud ingest actions with regular expression, index string, and Apply button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/4_4.png?raw=true)

7. Click **Save** to save and deploy the new rule.

## Task 5: Check Your Work so Far

Now that the modifications have been deployed, it's time to check to make sure the ingest actions rulesets are applying like they're intended to be.  If at any point the results from these searches don’t match up with what you see in your Splunk Cloud environment, we recommend checking over the relevant task first before asking for assistance.

1. For the `cisco:asa` events, compare the results between the two searches below:

```
index=firewall sourcetype=cisco:asa
```

```
index=firewall sourcetype=cisco:asa 106023 OR 302020 OR 302021 OR 313005 OR 106015 OR 106006 OR 106001 OR 302022
OR 302023 OR 302024 OR 302025 OR 302026 OR 302027 OR 4001*
```

Notice that recent events exist for the first search, but not the second.  This indicates that ingest actions ruleset is correctly filtering out ICMP-related messages.

![Splunk Cloud search results of index=firewall sourcetype=cisco:asa with the search highlighted in red and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/5_1.png?raw=true)

![Splunk Cloud search results of index=firewall sourcetype=cisco:asa 106023 OR 302020 OR 302021 OR 313005 OR 106015 OR 106006 OR 106001 OR 302022OR 302023 OR 302024 OR 302025 OR 302026 OR 302027 OR 4001* with the search highlighted in red and search results & timeline with missing events highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/5_2.png?raw=true)

2. Similarly with the eStreamer logs, run the search below:

```
index=firewall sourcetype=cisco:estreamer:data
```

Notice how the recent events are smaller now, and any key/value pairs with `N/A`, `null`, or `0`s have been removed from the event.

![Splunk Cloud search results of index=firewall sourcetype=cisco:estreamer:data with the search highlighted in red and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/5_3.png?raw=true)

3. To check the credit card data redaction, run the following search:

```
index=purchases sourcetype=web_purchases
```

Notice that the `credit_card` values are now redacted, and that only the last 4 digits of the credit card number are in the events.

![Splunk Cloud search results of index=purchases sourcetype=web_purchases with the search highlighted in red redacted credit card numbers in the events highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/5_3.png?raw=true)

4. Check that there are OneDrive-related events being routed to the `storage_services` index by running the search below:

```
index=storage_services sourcetype=o365:management:activity
```

![Splunk Cloud search results of index=storage_services sourcetype=o365:management:activity with the search highlighted in red and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/5_5.png?raw=true)

5. [Admin’s Little Helper for Splunk](https://splunkbase.splunk.com/app/6368) is an application that allows admins to view on-disk configured via Splunk searches.  Using [Admin’s Little Helper for Splunk](https://splunkbase.splunk.com/app/6368), view the configurations on-disk by running the search below:

```
| btool props list --debug | search */local/props.conf RULESET | dedup btool.stanza
```

![Splunk Cloud search results of | btool props list --debug | search */local/props.conf RULESET | dedup btool.stanza with the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/5_6.png?raw=true)

## Task 6: Route Data to Amazon S3

The data science team is requesting that AWS VPC Flow Logs be sent to their data lake, which is in Amazon S3.  Coincidentally, the compliance team would also like these logs to be sent to the same Amazon S3 bucket for long-term retention.  In this task, you'll configure an Amazon S3 destination for Splunk to use, then an ingest actions ruleset for Splunk to send these events to an Amazon S3 bucket by using ingest actions.  

Any AWS-side components for this task including the S3 bucket and the bucket policy have already been configured for you in this workshop.

1. Run the search below to view the AWS VPC Flow Log data:

```
index=aws sourcetype=aws:cloudwatchlogs:vpcflow
```

![Splunk Cloud search results of index=aws sourcetype=aws:cloudwatchlogs:vpcflow with the search highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/6_1.png?raw=true)

2. Launch the new destination wizard in Ingest Actions by clicking **Settings** > **Ingest Actions** > **Destinations** > **New Destination**

![Splunk Cloud ingest actions page with the Destinations tab and Create new Destination text highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/6_2.png?raw=true)

3. Configure the first set of destination settings by filling out the following fields in the `Create New Destination: Step 1` pane:
	- **Destination Title:** `ia-dest`
	- **S3 Bucket Name:** add `ia-dest` to the end of the pre-populated text.  When sending to an S3 destination using ingest actions, the bucket name must start with the Splunk Cloud stack name, and the randomly defined prefix.
4. Click the **Next** button.

![Splunk Cloud ingest actions new destination page 1 with the Destination Title, S3 Bucket Name, and Next button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/6_3.png?raw=true)

5. Finish configuring the new destination by clicking the **Generate Bucket Policy** button, then clicking the **Test Connection** button, and finally clicking **Save** to add the new destination.

If everything is configured correctly, the test should come back indicating it was successful.

![Splunk Cloud ingest actions new destination page 2 with the Generate Bucket Policy, Test Connection,and Save buttongs highlighted in red, and test results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/6_4.png?raw=true)

6. Create a new ruleset by clicking the **Rulesets** tab then clicking the **New Ruleset** button.

![Splunk Cloud ingest actions page showing previously configured rulesets for this module, with Rulesets tab and New Ruleset button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/6_5.png?raw=true)

7. Enter a descriptive **Ruleset Name**, such as `awsvpcflowlog-routing` to indicate that this ruleset applies to AWS VPC Flow Logs,  and that this ruleset is intended to route data.
8. Select the `aws:cloudwatchlogs:vpcflow` sourcetype by clicking the **Select sourcetype...** box, typing `aws:cloudwatchlogs:vpcflow`, and clicking **Use aws:cloudwatchlogs:vpcflow**.  
9. Populate the `Data Preview` pane by clicking the **Sample** button.
10. Create a new rule by clicking the **+ Add Rule**, and selecting the **Route to Destination** rule.
11. Configure the rule by clicking in the field just below **Immediately send to** and selecting `ia-dest` to add the S3 bucket destination you just added to the destinations the AWS VPC Flow Logs will be sent to.
12. Save and deploy the rule by clicking **Apply** then **Save**.

![Splunk Cloud ingest actions ruleset page with Route to Destination rule being configured with ruleset and Apply and Save buttons highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/6_6.png?raw=true)

At this point, `aws:cloudwatchlgos:vpcflow` data will start being sent to the S3 bucket.  You'll be using AWS VPC Flow Log data that has been routed to Amazon S3 via ingest actions in the next module.

## (Bonus) Task 7: Reverse Regex Golf

You may have noticed the regex in task 2 was not the prettiest: `(([a-z0-9\_]+=(""|(b')?[0-]+(')?|TLS_NULL_WITH_NULL_NULL|Unknown|"Not Checked"|[0:]+|N\/A)\s))` .  

On regex101.com, with PCRE2 selected, it takes 7067 steps to process the sample event below.

```
rec_type=71 event_sec=1684439277 app_proto=Unknown client_app=Unknown client_version="" url="" connection_id=60183 dest_autonomous_system=0 dest_mask=0 dest_tos=0 dns_query="" dns_rec_id=0 dns_resp_id=0 dns_ttl=0 iface_egress=Inside.71 sec_zone_egress=INside file_count=0 first_pkt_sec=1684439333 http_referrer="" http_response=0 iface_ingress=Outside.72 sec_zone_ingress=OUTside src_ip_country="united states" src_ip=210.141.241.173 src_port=52620 src_bytes=62 src_pkts=1 instance_id=4 ips_count=0 num_ioc=0 src_host=:: last_pkt_sec=1684439423 monitor_rule_1=N/A monitor_rule_2=N/A monitor_rule_3=N/A monitor_rule_4=N/A monitor_rule_5=N/A monitor_rule_6=N/A monitor_rule_7=N/A monitor_rule_8=0 netbios_domain="" netflow_src=00000000-0000-0000-0000-000000000000 fw_policy=00000000-0000-0000-0000-000064620104 ip_proto=TCP referenced_host="" dest_ip_country="united states" dest_ip=71.60.189.16 dest_port=51401 dest_bytes=0 dest_pkts=0 fw_rule_action=Block fw_rule=0 fw_rule_reason="IP Block" security_context=b'00000000000000000000000000000000' ip_layer=0 sec_intel_list1=Malware sec_intel_list2=0 sec_intel_ip=Source sinkhole_uuid=00000000-0000-0000-0000-000000000000 snmp_in=0 snmp_out=0 src_autonomous_system=0 src_mask=0 src_tos=0 ssl_actual_action=Unknown ssl_cert_fingerprint=b'0000000000000000000000000000000000000000' ssl_cipher_suite=TLS_NULL_WITH_NULL_NULL ssl_expected_action=Unknown ssl_flow_error=0 ssl_flow_flags=0 ssl_flow_messages=0 ssl_flow_status=Unknown ssl_policy_id=b'00000000000000000000000000000000' ssl_rule_id=0 ssl_server_cert_status="Not Checked" ssl_server_name="" ssl_session_id=b'0000000000000000000000000000000000000000000000000000000000000000' ssl_ticket_id=b'0000000000000000000000000000000000000000' ssl_url_category=0 ssl_version=Unknown tcp_flags=0 url_category=Unknown url_reputation=Unknown user_agent="" user=Unknown vlan_id=0 web_app=Unknown event_usec=0 event_subtype=1 event_type=1003 mac_address=00:00:00:00:00:00 rec_type_simple=RNA rec_type_desc="Connection Statistics" sec_intel_event=Yes sensor=LSS-SM1 event_desc="Flow Statistics"
```

![Regex101.com page shown with regular expression and test string highlighted in red, and the matches & steps highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_1/7_1.png?raw=true)

With normal regex golf you want to write the most efficient regex.  But this is _reverse_ regex golf!

Using the sample event and a limit of 200 characters for the regex, how inefficient can you make the regex while still matching everything the original regex matches?  We’ll measure inefficiency as the number of steps, so the higher the step count the better!
