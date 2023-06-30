# Module 4: DDSS Rehydration and Searching

In this module, you'll be rehydrating DDSS data, searching it from Splunk Enterprise, and making it available to be searched from Splunk Cloud through Splunk-to-Splunk Federated Search.

As a reminder, DDSS is a storage tier where data that ages out of Splunk Cloud is exported to a customer-owned Amazon S3 or Google Cloud Platform Google Cloud Storage (GCS) account.  With DDSS data, the customer is directly responsible for any data storage or rehydration fees.  Data sent to a DDSS destination needs to be "rehydrated" by Splunk before it is searchable, just like other data frozen by Splunk Enterprise.

The sales team has requested that the data being sent to the DDSS destination from Module 3 be made searchable so that they can search it from Splunk Cloud.  Specifically for this module, you will be:
1. Downloading & rehydrating DDSS data from an Amazon S3 bucket
2. Configuring Federated Search so that the Splunk Cloud environment can search the rehydrated data

## Task 1: Download & Rehydrate DDSS Data From an Amazon S3 Bucket

In this task, you'll be using the Splunk Enterprise host to download and rehydrate DDSS data.

1. In a terminal window, or your preferred SSH client, open an SSH session to the *Splunk Enterprise* host using the provided host name on port 2222.  Utilize the username and password provided by Splunk Show.
2. Create a folder to place the DDSS data by running the following command:

```
sudo -u splunk mkdir /opt/splunk/ddss-rehydrate/
```


3. Download the staged DDSS data from the Amazon S3 bucket by running the following command:

```
sudo -u splunk aws s3 sync s3://pla1750b-public-ddss-s3-bucket/ /opt/splunk/ddss-rehydrate/
```

4. Rehydrate all of the downloaded files from the Amazon S3 bucket that belong to the `purchases` index by running this command:

```
for d in /opt/splunk/ddss-rehydrate/purchases/*; do sudo -u splunk /opt/splunk/bin/splunk rebuild "$d" ; done
```

It's ok if error messages are logged specifying that the index doesn't exist.

5. Create a new index named `ddss-rehydrate` to hold this data, specifying `/opt/splunk/ddss-rehydrate/purchases` as the `thawdPath`.  The easiest way to do this is to run the command below

```
echo -e "[ddss-rehydrate]\nhomePath = /opt/splunk/var/lib/splunk/$_index_name/db\ncoldPath = /opt/splunk/var/lib/splunk/$_index_name/colddb\nthawedPath = /opt/splunk/ddss-rehydrate/purchases\nfrozenTimePeriodInSecs = 4294967295\nmaxTotalDataSizeMB = 4294967295" | sudo -u splunk tee /opt/splunk/etc/system/local/indexes.conf > /dev/null
```

6. Restart Splunk by running the following command

```
sudo systemctl restart Splunkd
```

7. Navigate to the Splunk Enterprise URL provided by Splunk Show and log in as the `admin` user with the password provided by Splunk Show.
8. To verify the data rehydrated successfully and is searchable, verify that the following search returns results for `All time`:

```
index=ddss-rehydrate sourcetype=web_purchases
```


The DDSS data has been successfully rehydrated!  You as the Splunk admin, or anyone who has access to the `ddss-rehydrate` on the Splunk Enterprise host can now search the DDSS data.  However, the sales team wants to be able to search this data from Splunk Cloud.


## Task 2: Configure Splunk-to-Splunk Federated Search

In this task, you'll be configuring Splunk-to-Splunk federated search (S2S-FS) so that the Splunk Cloud environment can search the rehydrated data on the Splunk Enterprise host.  

Normally, you'd want to use a dedicated service account just for S2S-FS on the remote search head (the Splunk Enterprise host in this workshop) for security reasons and to granularly-control what access the federated search head (Splunk Cloud in this workshop) has access too.  However, for this workshop you'll be using the `admin` account on the Splunk Enterprise host.

1. Navigate back to the Splunk Cloud host.
2. Open the Federated search settings page by navigating to **Settings** > **Federated search**
3. Start the process of adding a new Federated Provider by clicking the **Add federated provider** button, then selecting **Splunk**
4. Click the **Next** button.
5. Configure the S2S-FS Provider by filling out the following fields:
	1. **Provider name**: `pla1750b-s2s-fs`
	2. **Remote host**: This will be the hostname of the Splunk Enterprise host, provided by Splunk Show, specifying port 8089.  As an example, if the Splunk Show hostname is `i-00be76940de00b821.splunk.show`, this setting should be `i-00be76940de00b821.splunk.show:8089`.  See the screenshot below for an example.
	3. **Service account username**: `admin`
	4. **Service account password**:  This is the `admin` password for the Splunk Enterprise host provided by Splunk Show
6. Finish configuring the S2S-FS  Provider by clicking the **Warning and consent** checkmarkbox
7. Verify the configuration is correct by clicking the **Test connection** button
8. If the test is successful, save the configuration by clicking the **Save** button.  If the test is not successful, verify the configuration from step 5 above.
9. Start the process of adding a S2S-FS federated index by clicking the **Federated Indexes** tab and clicking the **Add federated index** button.
10. Fill out the following fields:
	1. **Federated index name**: `pla1750b-s2s-fs-ddss-rehydrate`
	2. **Federated provider**: `pla1750b-s2s-fs (Splunk)`
	3. **Remote dataset Dataset type**: `Index`
	4. **Remote dataset Dataset name**: `ddss-rehydrate`
11. Finish configuring the federated index by clicking the **Save** button.
12. Run the following search over `All time` to verify the S2S-FS configuration is correct:

```
index=federated:pla1750b-s2s-fs-ddss-rehydrate sourcetype=web_purchases
```

That's it!  The rehydrated DDSS data is now available for searching using the Splunk Cloud environment.  From here, you would grant the sales team access to the `federated:pla1750b-s2s-fs-ddss-rehydrate` index for them to search the rehydrated DDSS data.  Once they finished, you would remote the federated index and federated search provider in Splunk Cloud, then shut down and/or terminate the Splunk Enterprise host.