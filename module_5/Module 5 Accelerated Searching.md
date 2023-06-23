# Module 5: Accelerated Searching

Accelerated searching is a great way to reduce search runtime by pre-computing (basically pre-running) frequently searched data.  There are different ways to get value from pre-computing these results and different ways to store these results depending on how they’ll get used.  In this module, you will explore a few different options.  You'll work with storing summarized data in a lookup table first, then storing data in a summary index, and wrap up with metricizing events to store in a metrics index.

## Task 1: Store Summarized Login Information in a Lookup Table

The SOC is interested in using machine learning to help detect when an anomalous amount of data is being downloaded or uploaded to OneDrive.  They’d like to use outlier detection to accomplish this, but in order to do this they want to search off of the past 90 days of OneDrive data every hour to figure out a baseline for what is a normal amount of data transfer.  Searching over the past 90 days of OneDrive data every hour to detect anomalies will put an unnecessary amount of search load on Splunk and there’s a better way: summary indexing!

In this task you'll configure a search to gather metrics related to OneDrive usage, and save those results to a lookup table.  This lookup table can then be used to establish a baseline for what "normal" OneDrive usage looks like, and used in an alert instead of searching over the raw 90 days of data every time the search powering the alert needs to run.

1. In module 1, you redirected the OneDrive-related events to go to the `storage_services` index, however for this task we'll want to reference as much data as possible, so you'll be searching against OneDrive data in both the `main` and `storage_services` index.  Run the search below to make sure data is getting pulled back for the whole 30 minutes:

```
index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive
```

![Splunk Cloud search results of index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive with search highlighted in red and "Workload: OneDrive" highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/1_1.png?raw=true)

2. Add the timechart command onto the search to see the count per-minute for each Operation.  Just for testing purposes (and this workshop), you'll be looking at the data with a 1-minute granularity (span), but if you were deploying this for the SOC you'd use a 1-hour granularity (span).

```
index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive
| timechart span=1m count by Operation
```

![Splunk Cloud search results of index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive | timechart span=1m count by Operation with search highlighted in red and tabled results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/1_2.png?raw=true)


3. Create a new lookup named `onedrive-activity.csv` by adding the `outputlookup` command to the saerch:

```
index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive
| timechart span=1m count by Operation
| outputlookup onedrive-activity.csv
```

This search can be saved and configured to run as a saved search once-a-day so that the SOC's anomaly detection search needs to search has up-to-date information via the lookup table to reference, as opposed to searching the raw data every time the alert needs to run.  In the real world (outside of the workshop), the timeframe for the search would be changed to something more substantial (ex looking at the past 90 days as opposed to 30 minutes), and the span would be updated to match with the SOC's alert (ex `span=1h` instead of `span=1m`).

![Splunk Cloud search results shown with search index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive | timechart span=1m count by Operation | outputlookup onedrive-activity.csv highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/1_8.png?raw=true)

4. Make sure the lookup table was created correctly by verifying this search below, which dumps the contents of the lookup table:

```
| inputlookup onedrive-activity.csv
```

![Splunk Cloud search results for | inputlookup onedrive-activity.csv shown with search highlighted in red, and results table highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/1_7.png?raw=true)

Now that the lookup table has been created, the search that created the lookup table can be saved and configured to run automatically to update the lookup table using the `outputlookup` command.

5. The data from the lookup table can be appended to a search using the `append` and `inputlookup` commands.  Set the time range picker to **1 minute** and run the search below to see this in action:

```
index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive 
| timechart span=1m count by Operation 
| append 
    [| inputlookup onedrive-activity.csv]
```

Notice how that even though the search is searching for 1 minute of results (as set by the time picker), there are more than 1 minute of results.  The other results are being populated by the lookup table.

![Splunk Cloud search results shown with search index=storage_services OR index=main sourcetype=o365:management:activity Workload=OneDrive  | timechart span=1m count by Operation  | append  [| inputlookup onedrive-activity.csv] highlighted in red and result times highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/1_9.png?raw=true)

## Task 2: Summarize Purchase Records

The sales department would like to keep summarized, statistical data on product purchases long-term.  Ideally, these summarized events would be kept around for seven years, which is significantly over the retention period of the purchasing events in the punches index.  The sales department has explained they don't need the raw purchase records in Splunk long-term, just the summarized data.  

In this task, you'll be configuring summary indexing to store this data long-term.

You’ll be using the `collect` command and summary indexing in this task.  Here’s a few tips/tricks to using collect to help with summary indexing.
- It's considered a best practice to use a transforming command (like `timechart` or `stats`) to aggregate the data before sending it to the summary index.
- The `marker` option in the `collect` command can be used to add additional fields to help describe the summarized data.
- We _highly_ recommend versioning the data being sent to the summary index in case you need to iterate on the search generating the data to be summarized or the collect command itself.  We’ll be doing that in this task using the `marker` option.
- Use the `testmode` option for the collect command when writing and testing the search to avoid writing data to the destination index.
- By default, `collect` will set the sourcetype to `stash`, which will not impact license utilization.  If you set the sourcetype in the `collect` command to something other than `stash`, this will impact license usage.
- Use a `marker` of `search_name` to make it easy to figure out what search the summarized data came from 
- Make the search summarizing the data performance to minimize Splunk resource (and SVC) utilization.

Follow these instructions to configure the summary indexing:

1. Find the data you'll be summarizing by running the search below

```
index=purchases sourcetype=web_purchases
```

![Splunk Cloud search results shown with search of index=purchases sourcetype=web_purchases highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/2_1.png?raw=true)

2. The sales team is only interested in knowing about purchases, so filter for just those records

```
index=purchases sourcetype=web_purchases action=purchase
```

![Splunk Cloud search results shown with search of index=purchases sourcetype=web_purchases action=purchase highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/2_2.png?raw=true)

3. The sales team would like total revenue (cost of the product) per-product.  They'd like the data aggregated by day, but for testing purposes (and this workshop), use 1-minute granularity

```
index=purchases sourcetype=web_purchases action=purchase
| timechart span=1m sum(cost) as revenue by product
```

![Splunk Cloud search results shown with search of index=purchases sourcetype=web_purchases action=purchase | timechart span=1m sum(cost) as revenue by product highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/2_3.png?raw=true)

4. Add the `collect` command next.  The search below will take the results from the `timechart` command, send the data to the `summary` index, with a markers of `version=100`, & `search_name=summarize_purchases`.  Set `testmode` on this search to view the results without sending any data to the summary index.

```
index=purchases sourcetype=web_purchases action=purchase
| timechart span=1m sum(cost) as revenue by product
| collect index=summary source=summarize_purchases marker="version=100, search_name=summarize_purchases" testmode=true
```

If you zoom into the screenshot (or the results on your screen), you can see the data that will be sent to the summary index in the `_raw` field.

Notice how the data is dropped into key-value format with each filed from the lookup table being a field, and the value from the table being a value for that key.

![Splunk Cloud search results shown with search of index=purchases sourcetype=web_purchases action=purchase | timechart span=1m sum(cost) as revenue by product | collect index=summary source=summarize_purchases marker="version=100, search_name=summarize_purchases" testmode=true highlighted in red and raw column in results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/2_4.png?raw=true)

5. At this point, the data looks good, so the scheduled search can be created without the `testmode=true` option.   However, since this is a workshop, run the searches below to verify send data to the summary index then verify the data can be searched back:

```
index=purchases sourcetype=web_purchases action=purchase
| timechart span=1m sum(cost) as revenue by product
| collect index=summary source=summarize_purchases marker="version=100, search_name=summarize_purchases"
```

```
index=summary version=100
| table *
```

Notice how the data can be sent to a table just like if the original `timechart` command were run.  This summarized data can be kept inside of the `summary` index and since it'll take up less space it'll cost less to store as compared to storing the raw events.

![Splunk Cloud search results shown with search of index=purchases sourcetype=web_purchases action=purchase | timechart span=1m sum(cost) as revenue by product | collect index=summary source=summarize_purchases marker="version=100, search_name=summarize_purchases" highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/2_5.png?raw=true)

![Splunk Cloud search results shown with search of index=summary version=100 | table * highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/2_6.png?raw=true)

6. If you're interested in determining how much less space the summarized events take up, run the search below:

```
(index=purchases sourcetype=web_purchases action=purchase) OR (index=summary version=100) 
| eval eventSize=len(_raw)
| stats sum(eventSize) as total_event_size by index
```

Notice how the event sizes per index are significantly different, with the summarized data in the `summary` index is smaller than the raw events in the `purchases` index take up more space.

## Task 3: Metricize Events

Similarly, the networking team would like to keep aggregated statistics long-term on VPC Flow Logs.  Ideally, this data would be metricized and stored as metric-style events to minimize data storage costs and maximize query performance.  The networking team would like to store how many bytes were transmitted to/from each ENI in each AWS account at a 1-minute granularity, then keep this data for 365 days.  They’d like this data placed in the `summary_metrics` index.

1. Run the search below to find the VPC Flow log data that will be metricized

```
index=aws sourcetype=aws:cloudwatchlogs:vpcflow
```

![Splunk Cloud search results shown with search of index=aws sourcetype=aws:cloudwatchlogs:vpcflow highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/3_1.png?raw=true)

2. Since the `timechart` command has a limitation of only being able to group by 1 field, use a combination of the `bin` and `stats` commands to aggregate the data on a 1-minute granularity by the ENI ID, AWS account number, and time (or `interface_id`, `account_id`, and `_time` respectively).

```
index=aws sourcetype=aws:cloudwatchlogs:vpcflow
| bin span=1m _time
| stats sum(bytes) as sum_bytes by interface_id, account_id, _time
```

![Splunk Cloud search results shown with search of index=aws sourcetype=aws:cloudwatchlogs:vpcflow | bin span=1m _time | stats sum(bytes) as sum_bytes by interface_id, account_id, _time highlighted in red, and results in table form highlighted ing reen](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/3_2.png?raw=true)

3. Use the rename command to properly format the `sum_bytes` field into a metric name format
```
index=aws sourcetype=aws:cloudwatchlogs:vpcflow
| bin span=1m _time
| stats sum(bytes) as sum_bytes by interface_id, account_id, _time
| rename sum_bytes as metric_name:sum_bytes
```

![Splunk Cloud search results shown with search of index=aws sourcetype=aws:cloudwatchlogs:vpcflow | bin span=1m _time | stats sum(bytes) as sum_bytes by interface_id, account_id, _time  | rename sum_bytes as metric_name:sum_bytes highlighted  in red, and results in table form highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/3_3.png?raw=true)

4. Now, use the `mcollect` command to send the metricized events to the `summary_metrics` index, specifying `account_id` and `interface_id` as metric dimensions, with a version of `100` using a marker & `search_name` of `summarize_vpcflow` using markers:

```
index=aws sourcetype=aws:cloudwatchlogs:vpcflow
| bin span=1m _time
| stats sum(bytes) as sum_bytes by interface_id, account_id, _time
| rename sum_bytes as metric_name:sum_bytes
| mcollect index=summary_metrics source=aws-vpcflow marker="version=100, search_name=summarize_vpcflow" account_id, interface_id
```

![Splunk Cloud search results shown with search of index=aws sourcetype=aws:cloudwatchlogs:vpcflow | bin span=1m _time | stats sum(bytes) as sum_bytes by interface_id, account_id, _time | rename sum_bytes as metric_name:sum_bytes | mcollect index=summary_metrics source=aws-vpcflow marker="version=100, search_name=summarize_vpcflow" account_id, interface_id highlighted  in red, and results in table form highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/3_4.png?raw=true)


5. You can check your work by using the `mpreview` command:

```
| mpreview index=summary_metrics filter=version=100
```

![Splunk Cloud search results shown with search of | mpreview index=summary_metrics filter=version=100 highlighted  in red, and results in table form highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/v1/module_5/3_5.png?raw=true)

From here, a saved search can be created to regularly (on a cadence) metricize the events and send them to the `summary_metrics` index.  The networking team can then search the metricized events using the `mstats` command.