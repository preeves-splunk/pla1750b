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
sudo mkdir /opt/splunk/ddss-rehydrate/
```

3. Download the staged DDSS data from the Amazon S3 bucket by running the following command:

```
sudo AWS_ACCESS_KEY_ID=AKIASBDOGC2FVLY4SENQ AWS_SECRET_ACCESS_KEY=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 aws s3 sync s3://pla1750b-public-ddss-s3-bucket/ /opt/splunk/ddss-rehydrate/
```

Note: We know that hard-coding credentials like this into a command, especially in a public GitHub repo is a security worse-practice and not recommended.  We're doing this to get around AWS cross-account limits.  Literally the only thing this user can do is list and get objects from the public bucket that holds DDSS data.

![Terminal window with results of aws s3 sync command downloading files via sudo aws s3 sync s3://pla1750b-public-ddss-s3-bucket/ /opt/splunk/ddss-rehydrate/](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/1_1.png?raw=true)

4. Rehydrate all of the downloaded files from the Amazon S3 bucket that belong to the `purchases` index by running this command:

```
for d in /opt/splunk/ddss-rehydrate/purchases/*; do sudo /opt/splunk/bin/splunk rebuild "$d" ; done
```

This command might take a few minutes to complete.  Also, it's ok if error messages are logged specifying that the index doesn't exist.

![Terminal window with results of splunk rebuild command for d in /opt/splunk/ddss-rehydrate/purchases/*; do sudo /opt/splunk/bin/splunk rebuild "$d" ; done](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/1_2.png?raw=true)

5. Create a new index named `ddss-rehydrate` to hold this data, specifying `/opt/splunk/ddss-rehydrate/purchases` as the `thawdPath`.  The easiest way to do this is to run the command below

```
echo -e "[ddss-rehydrate]\nhomePath = /opt/splunk/var/lib/splunk/$_index_name/db\ncoldPath = /opt/splunk/var/lib/splunk/$_index_name/colddb\nthawedPath = /opt/splunk/ddss-rehydrate/purchases\nfrozenTimePeriodInSecs = 4294967295\nmaxTotalDataSizeMB = 4294967295" | sudo tee /opt/splunk/etc/system/local/indexes.conf > /dev/null
```

6. Restart Splunk by running the following command

```
sudo /opt/splunk/bin/splunk restart
```

![Terminal window with results of restarting splunk using sudo /opt/splunk/bin/splunk restart](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/1_3.png?raw=true)

7. Navigate to the Splunk Enterprise URL provided by Splunk Show and log in as the `admin` user with the password provided by Splunk Show.
8. To verify the data rehydrated successfully and is searchable, verify that the following search returns results for `All time`:

```
index=ddss-rehydrate sourcetype=web_purchases | head 100
```

![Splunk Enterprise window showing results of index=ddss-rehydrate sourcetype=web_purchases | head 100 with search & all time highlighted in red and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/1_2.png?raw=true)

The DDSS data has been successfully rehydrated!  You as the Splunk admin, or anyone who has access to the `ddss-rehydrate` on the Splunk Enterprise host can now search the DDSS data.  However, the sales team wants to be able to search this data from Splunk Cloud.

## Task 2: Configure Splunk-to-Splunk Federated Search

In this task, you'll be configuring Splunk-to-Splunk federated search (S2S-FS) so that the Splunk Cloud environment can search the rehydrated data on the Splunk Enterprise host.  

Normally, you'd want to use a dedicated service account just for S2S-FS on the remote search head (the Splunk Enterprise host in this workshop) for security reasons and to granularly-control what access the federated search head (Splunk Cloud in this workshop) has access too.  However, for this workshop you'll be using the `admin` account on the Splunk Enterprise host.

1. Navigate to the Splunk Cloud host and log back in if necessary, using the provided username and password from Splunk Show.
2. Open the Federated search settings page by navigating to **Settings** > **Federated search**

![Splunk Cloud settings page with Settings and Federated search highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_1.png?raw=true)

3. Start the process of adding a new Federated Provider by clicking the **Add federated provider** button.

![Splunk Cloud Federated search settings page with Add federated provider button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_2.png?raw=true)

4. Select **Splunk**, and click the **Next** button.

![Splunk Cloud Add Federated Provider pane shown with Splunk option and Next button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_3.png?raw=true)

5. Configure the S2S-FS Provider by filling out the following fields:
	1. **Provider name**: `pla1750b-s2s-fs`
	2. **Remote host**: This will be the hostname of the Splunk Enterprise host, provided by Splunk Show, specifying port 8089.  As an example, if the Splunk Show hostname is `i-024cab16d43fcecb9.splunk.show`, this setting should be `i-024cab16d43fcecb9.splunk.show:8089`.  See the screenshot below for an example.
	3. **Service account username**: `admin`
	4. **Service account password**:  This is the `admin` password for the Splunk Enterprise host provided by Splunk Show
	5. **Warning and consent**: checked

![Splunk Cloud Add Federated Provider pane with settings mentioned above highlighted in red and numbered with red numbers](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_4.png?raw=true)

6. Verify the configuration is correct by clicking the **Test connection** button
7. If the test is successful, save the configuration by clicking the **Save** button.  If the test is not successful, verify the configuration from step 5 above.

![Splunk Cloud Add Federated Provider with Test connection and save buttons highlighted in red, and Connection successful message highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_5.png?raw=true)

8. Start the process of adding a S2S-FS federated index by clicking the **Federated Indexes** tab and clicking the **Add federated index** button.

![Splunk Cloud Federated search settings page with Federated Indexes tab and Add federated index buttons highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_6.png?raw=true)

9. Fill out the following fields:
	1. **Federated index name**: `pla1750b-s2s-fs-ddss-rehydrate`
	2. **Federated provider**: `pla1750b-s2s-fs (Splunk)`
	3. **Remote dataset Dataset type**: `Index`
	4. **Remote dataset Dataset name**: `ddss-rehydrate`
10. Finish configuring the federated index by clicking the **Save** button.

![Splunk Cloud create a Federated Index pane open with options above highlighted in red with red numbering to indicate which options need to be set and where, and Save button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_7.png?raw=true)

11. Run the following search over `All time` to verify the S2S-FS configuration is correct:

```
index=federated:pla1750b-s2s-fs-ddss-rehydrate sourcetype=web_purchases
```

![Splunk Cloud search results of index=federated:pla1750b-s2s-fs-ddss-rehydrate sourcetype=web_purchases with search, time range, and search button highlighted in red, and search results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_4/2_7.png?raw=true)

That's it!  The rehydrated DDSS data is now available for searching using the Splunk Cloud environment.  From here, you would grant the sales team access to the `federated:pla1750b-s2s-fs-ddss-rehydrate` index for them to search the rehydrated DDSS data.  Once they finished, you would remote the federated index and federated search provider in Splunk Cloud, then shut down and/or terminate the Splunk Enterprise host.