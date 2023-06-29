# Module 0: Introduction 

This module is designed to help you get acquainted with the environment that we'll be using during this workshop.  It's not meant to be challenging and really just designed to make sure you can log into everything before the next module.  If you run into problems or need help please ask!

Congratulations on your new job as Splunk Cloud administrator at Frothly!  Let's get you acquainted with your new environment by logging in and seeing what data is available.  We'll get started with the Splunk Cloud environment which is ingesting a variety of data, then look at the on-prem Splunk Enterprise environment that's only used for rehydrating and searching DDSS data.

A few notes on how this lab is formatted:
- Mono-spaced text `like this` and code blocks indicate specific callouts you need to type or pay attention to
- Bolded text **like this** indicates an action you need to take such as clicking a button in a browser window
- All of our screenshots throughout the lab are taken in the "Light" theme so it's easier to read, but your environment may use a "Dark" theme as that is the new system default.
- The red boxed sections in the screenshots indicate where you need to interact with the browser window or terminal session by clicking or typing something.
- The green boxed sections in the screenshots indicate something you should read, view, or note.
- Both the Splunk Cloud and Splunk Enterprise hosts are configured to use the light theme so that what you see in the screenshots matches what you see on your screen.  However if you'd like to switch your interface to use the dark theme you're welcome to do so.

## Task 1: Explore Splunk Cloud Environment

1. Open a browser tab and navigate to the Splunk Cloud URL provided by Splunk Show.  Log in as the `sc_admin` user and provided password provided by Splunk Show.  If prompted, click the **I accept these terms** checkmark box, then the Ok button.

![Splunk Cloud login prompt with login fields highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_1.png?raw=true)

![Splunk Cloud Terms of Service screen with I accept these terms and Ok button highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_2.png?raw=true)

2. To discover what data sourcetypes and indexes are available in the Splunk Cloud instance, navigate to the **Search & Reporting app** under the **Apps** menu in the upper left hand corner of your browser.  Utilize the following search by copying and pasting it into the Search field and clicking the magnifying glass to the right to execute the search.

![Splunk Cloud main screen with Search & Reporting app link highlighted in red](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_3.png?raw=true)

```
| tstats count where index=* by index, sourcetype
```

![Search results for | tstats count where index=* by index, sourcetype with search and search button highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_4.png?raw=true)

If there is data available, search through each of the following indexes in turn to look at the available data.  Note that each of the searches should return results from the past 10 minutes, but leave the time range picker set to **Last 30 Minutes**.

3. Search the `main` index:

```
index=main
```

![Splunk Cloud search results for index=main with search highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_5.png?raw=true)

4. Search the AWS VPC Flow Logs:

```
index=aws sourcetype=aws:cloudwatchlogs:vpcflow
```

![Splunk Cloud search results for index=aws \sourcetype=aws:cloudwatchlogs:vpcflow with search highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_6.png?raw=true)

5. Search the firewall events:

```
index=firewall
```

![Splunk Cloud search results for index=firewall with search highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_7.png?raw=true)

6. Search the purchase records:

```
index=purchases
```

![Splunk Cloud search results for index=purchases with search highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_8.png?raw=true)

## Task 2: Explore Splunk Enterprise Environment

1. In another browser tab, navigate to the Splunk Enterprise URL provided by Splunk Show and log in as the `admin` user with the password provided by Splunk Show.

2. While there currently aren't any indexes on this installation of Splunk yet, run the following search against the `_internal` index to ensure search is working properly.

```
index=_internal
```

![Splunk Enterprise search results for index=_internal with search highlighted in red, and results highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_9.png?raw=true)

3. In a terminal window, or your preferred SSH client, open an SSH session to the *Splunk Enterprise* host using the provided IP address.  Utilize the SSH username and password provided by Splunk Show.  Once you have logged in, validate that you are able to run commands utilizing `sudo` by executing `sudo whoami`.  The expected command response is `root`.

![Graphic showing terminal windo after SSH, with whoami results with root highlighted in green](https://github.com/preeves-splunk/pla1750b/blob/main/module_0/0_10.png?raw=true)