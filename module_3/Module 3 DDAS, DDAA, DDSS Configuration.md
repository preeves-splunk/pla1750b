# Module 3: DDAS, DDAA, DDSS Configuration

In this module you'll be creating some additional indexes in Splunk Cloud, and configuring data to be sent to different storage tiers after the data "ages out" of Splunk.  There are three different Splunk Cloud-based storage tiers you'll be working with in this module:

- Dynamic Data Active Searchable (DDAS): This is used for searchable data in Splunk Cloud.
- Dynamic Data Active Archive (DDAA): This is where non-searchable data is stored in Splunk Cloud that needs to be possibly be temporarily made searchable again in Splunk Cloud.  This is a great storage tier to use for any data that you don't want to delete because you want to easily restore it into Splunk Cloud for a period of time.
- Dynamic Data Self-Storage (DDSS): This is a storage tier where data that ages out of Splunk Cloud is exported to a customer-owned Amazon S3 or Google Cloud Platform Google Cloud Storage (GCS) account.  With DDSS data, the customer is directly responsible for any data storage or rehydration fees.   Data sent to a DDSS destination needs to be "rehydrated" by Splunk before it is searchable, just like other data frozen by Splunk Enterprise.  In module 4, you'll be walking through this process of downloading, rehydrating, and searching data from an Amazon S3 bucket with DDSS data sent to it.

Additional information about DDAS, DDAA, and DDSS storage can be found in the [Splunk Cloud Platform Service Description](https://docs.splunk.com/Documentation/SplunkCloud/latest/Service/SplunkCloudservice#Storage).

Specifically for this module and for Forthly, you will be:
1. Create a new index with DDAA storage enabled
2. Configure a DDSS destination
3. Configure an existing index to send to DDSS
4. Review DDAA restore process

## Task 1: Create a New Index

In this task you'll be creating a new index for the DevOps team.  The DevOps team has some new CI/CD-related data they'd like to ingest into Splunk Cloud, and they're requesting a new index to hold that data.  The DevOps team needs the data to be stored for a total of 1 year.  They'd like to be able to search the data for 90 days in Splunk Cloud, then have the data be sent to DDAA for an additional 275 days.  

The retention times for DDAS and DDAA storage are independent of each other, not additive, so we'll be configuring the DDAA tier to delete the data after the events are older than 365 days.

1. In Splunk Cloud, navigate to the index configuration page by clicking **Settings** > **Indexes**.

![Splunk Cloud Settings page with Settings and Indexes options highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/1_1.png?raw=true)


2. Start creating a new index by clicking the **New Index** button.

![Splunk Cloud Indexes Settings page with New Index button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/1_2.png?raw=true)

3. Configure the new `devops` index by filling out the following fields:
	- **Index name**: `devops`
	- **Max raw data size**: `0` GB (Setting this to `0` GB effectively means the data will be managed only by the retention period)
	- **Searchable retention (days)**: `90`
	- **Archive Retention Period**:  `365` days
4. Click the **Save** button

![Splunk Cloud Indexes Settings page with devops index name, 0GB, 90 days retention, 365 day retention for DDAA, and save button highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/1_3.png?raw=true)

5. If promtoed, click the **Continue** button

![Splunk Cloud showing Saving New Index Pane with contiue button highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/1_5.png?raw=true)

That's it!  The new `devops` will be deployed to the Splunk Cloud environment and ready for use within a few minutes.

## Task 2: Configure a DDSS Destination

In this task you'll be adding an Amazon S3-based destination for DDSS data to be sent to.  Since this is DDSS and not DDAA, the sales team has agreed to work with you to rehydrate this data if they ever need to search it.  Again, in module 4 you'll be walking through this process of downloading, rehydrating, and searching DDSS data.

Normally during this process the Splunk Cloud admin would work with an AWS administrator to create the necessary AWS-side components for a DDSS destination.  Any AWS-side components for this task including the S3 bucket and the bucket policy have already been configured for you in this workshop.

1. Navigate to the index configuration page by clicking **Settings** > **Indexes**.
2. Bring up the `purchases` configuration page by clicking **Edit** next to the `purchases` index.

![Splunk Cloud Indexes Settings page Edit link for purchases index highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/2_1.png?raw=true)

3. Open the `Self Storage Locations` page by clicking the `Self Storage` option, then clicking **Create a self storage location** .  This may open the page in a new tab or window, which is ok.

![Splunk Cloud Indexes Settings page for purchases index with Create a self storage link highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/2_2.png?raw=true)

4. Start creating the new DDSS destination by clicking the **New Self Storage Location** button.

![Splunk Cloud Self Storage Locations page with New Self Storage Location button highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/2_3.png?raw=true)

5. Start the configuration process for the DDSS destination by filling out the following fields:
	- **Title**: `ddss-dest`
	- **AWS S3 bucket name**: Copy and paste the quoted bucket name in the text below the bucket name, and add `ddss-dest` to the end of it.  See the screenshot below for an example of this.  The name of the Amazon S3 bucket must start with the Splunk Cloud environment name, and have the randomly-generated characters in that text in order to avoid naming collisions in AWS.

![Splunk Cloud New Self Storage Location Pane with title of ddss-dest, and AWS S3 bucket name of pla1750b-0603-qlfzgg20m0oa-ddss-dest for AWS S3 bucket name highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/2_4.png?raw=true)

6. Generate the Amazon S3 bucket policy by clicking the  **Generate** button.

Notice how Splunk Cloud generated and provided the Amazon S3 Bucket Policy that will need to be applied to the Amazon S3 bucket that will store the DDSS data.  Normally this would need to be shared with the AWS administrator so it can be applied to the destination Amazon S3 bucket.

![Splunk Cloud New Self Storage Location Pane with Generate button for AWS S3 bucket policy highlgihted in red, and bucket policy pane highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/2_5.png?raw=true)

7. Test the DDSS Destination configuration by clicking the **Test** button next to the "Test bucket policy" line.
8. If the test comes back a `Success!`, finish adding the DDSS destination by clicking the **Submit** button.  If the test does not come back successful, review the instructions from step 5, then try again, or ask for assistance.

![Splunk Cloud New Self Storage Location Pane with Test button for Test bucket policy and Submit button highlgihted in red, and Success! text highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/2_6.png?raw=true)


That's it!  The new DDSS destination will be added to the Splunk Cloud environment.

## Task 3: Configure an Index to Send to DDSS

Now that the DDSS destination has been added, you'll need to configure an index to send data to it.  The sales team would like all purchase-related data to be saved to an Amazon S3 bucket after it has been searchable in Splunk Cloud for 90 days.  The sales data is located in the `purchases` index. 

1. Navigate to the index configuration page by clicking **Settings** > **Indexes**.
2. Bring up the `purchases` configuration page by clicking **Edit** next to the `purchases` index.

![Splunk Cloud Indexes Settings page Edit link for purchases index highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/2_1.png?raw=true)

3. Configure this index to use the DDSS destination configured in Task 2 by selecting the `Self Storage` option, and selecting the Amazon S3 bucket name added in Task 2.
4. Finish configuring this index to use the DDSS destination by clicking the **Save** button.

![Splunk Cloud Indexes Settings page for purchases index with Self Storage option, Please select option dropdown, and pla1750b-0603-qlfzgg20m0oa-ddss-det options, and Save button highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/3_1.png?raw=true)

That's it!  After 90 days, the data from the `purchases` index will be sent to the DDSS destination bucket, just like the sales team requested.

## Task 4: (Bonus) Review DDAA Restore Process

This this task, you'll be reviewing how to temporarily retore data from DDAA so it's searchable.  In this task you'll just be reviewing screenshots and steps, not actually changing or clicking anything in the Splunk web interface.

Once requested, DDAA data is searchable within 24 hours of it being restored and is searchable for up to 30 days.  Also, you are only able to restore the equivelant of 10% of your DDAS storage entitlement from DDAA at a time.  Restored DDAA data can be forced to expire before the 30 days is over as well.

The steps below are just informational, you won't be following along in the web interface.  These instructions would be for restoring data from the `aws` index between June 4, 2023 and June 17th, 2023.

1. To start restoring data, you would navigate to **Settings** > **Indexes**, then click the **Restore** link next to the index you.

![Splunk Cloud Indexes Settings page with Restore link for aws index highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_1.png?raw=true)

2. You would then select the date range you need to to restore data from by modifying the two time range pickers in the `Restore Archive` pane to indicate the earliest and latest data to restore, and optionally an email address to be notified of updates on the restore task.
3. After selecting the date range to restore data from, you would click the **Check Size** button to verify that you have enough space to restore the data.
4. You would click the **Restore** button to start the restore process

Notice how in the screenshot below, the Total Restore Size of the data that will be restored is listed.

![Splunk Cloud Restore Archive pane for aws index shown with time range options of 6/4/2023 to 6/17/2023, Check Size and Restore buttons highlgihted in red, options from above numbered, and Total Restore size of 1.05GB highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_2.png?raw=true)

5. You would then click the **Restore** button on the `Confirmation` pop-up to start the restore process.

![Splunk Cloud Restore Archive Confirmation pop-up for restore job show with Restore button highlgihted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_3.png?raw=true)

6. At this point, Splunk Cloud will begin the restore process for the selected index on the selected times.  You can see the restore job information on the `Restore Archive` pane for this index now, with `Job Status` of `Pending`.  This job status will change as the restore job progresses.  Again, this restoration request will take about 24 hours to finish.

Notice how in the screenshot below you can now see the restore job status, and the job status.

![Splunk Cloud Restore Restore Archive pane for aws index shown with restore job information highlgihted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_4.png?raw=true)


7. After 24 hours the restore job has a status of `Success`:

![Splunk Cloud Restore Restore Archive pane for aws index shown with restore job information with status of Success highlgihted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_5.png?raw=true)

A screenshot showing a restoration request notification email for a similar DDAA restore job can be seen below:

![Screenshot of a DDAA restoration notificaiton email ](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_6.png?raw=true)

And data is searchable for the `aws` index for the requested time period:

![Splunk Cloud search results for index=aws | head 100 over the time frame of June 4th through June 23rd ](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_7.png?raw=true)

8. On the Restore Archive pane for the `aws` index, the restore job can also be expired manually, before the 30-day period is over.  This can be useful to free up space for additional data can be restored.

![Splunk Cloud Restore Restore Archive pane for aws index shown with restore job Clear link highlgihted in red ](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_8.png?raw=true)

Once the restore job has expired, either becuase the 30-day period has lapsed or the job was cleared manually before the 30-day period ended, the job status will change to `Cleared`:

![Splunk Cloud Restore Restore Archive pane for aws index shown with restore job Cleared status highlgihted in green ](https://github.com/preeves-splunk/pla1750b/blob/main/module_3/4_9.png?raw=true)

That's it!  Thanks for joining us on this optional review and tour of restoring DDAA data.