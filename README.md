Lambda Dashboard
====================

Lambda Dashboard is a Google Spreadsheet that provides two functions: 1) it can receive [Fluentd](http://www.fluentd.org/) event logs and display charts from them in real-time, 2) it can execute [Google BigQuery](https://developers.google.com/bigquery/?hl=ja) queries and display charts from the result every minute/hour. The Dashboard provides an easy way to build a simple [lambda architecture](http://lambda-architecture.net/) to get a merged view from real-time continuous query on streaming data and batch-based query on historical and large data set.

![Lambda Dashboard](http://i.giflike.com/sLgJtW9.gif)

Check out [Lambda Dashboard Demo Video](https://www.youtube.com/watch?v=EZkw5TDcCGw) to learn how it works.

## Features

- It's a **Google Spreadsheet**: hosted by Google at free, easy to customize and integrate with your business process even for non-programmers. Copy the spreadsheet, click some buttons as in Getting Started, and it's ready to use

- Easy **Real-time** analytics with CEP: Lambda Dashboard will update charts as soon as it receives logs from Fluentd. Useful for watching how the system stats and service KPIs are moving when you have game or campaign events etc. It also supports an easy integration with **[Norikra](https://github.com/norikra/norikra), a real-time CEP (Complex Event Processing)**, for sophisticated and complex real-time analytics with its [continuous query with EPL](http://esper.codehaus.org/tutorials/solution_patterns/solution_patterns.html).

- Easy **Big Data** analytics with BigQuery: you can execute BigQuery query just by entering a SQL on a sheet. The Dashboard will automatically execute it every minute/hour and draw a chart from the result. By combining the BigQuery with Norikra, you could have both real-time view and historical view of your big data on the same Dashboard

- Everything **Dockernized** for quick deployment: By using [Docker](http://docker.io/), it takes only 10 min to build and deploy the whole stack on Google Compute Engine instances

## Getting Started

To start using Lambda Dashboard, follow the instruction below.

1. Open [this spreadsheet](https://docs.google.com/spreadsheets/d/12LPtwVXNY85xGC86P1GcOWolAqAwkLAsuQoo9IIUqCo/edit) and select `File` - `Make a copy` menu to make a copy of it
1. Copy the URL of the copied spreadsheet to clipboard
1. Select `Tools` - `Script editor...` menu
1. On the Script editor, open `fluent_listener.gs`. Paste the copied URL on the place of `<<PLEASE PUT YOUR SPREADSHEET URL HERE>>`. Select `File` - `Save` menu to save the file

Now it's ready to use with Fluentd and/or BigQuery.

## Use with Fluentd for real-time analytics

To use the spreadsheet with Fluentd and Norikra for real-time analytics, follow the instruction below.

1. On the Script editor, select `Publish` - `Deploy as web app...`
1. On the `Deploy as web app` dialog, enter `1` in the `Project version` field and click `Save New Version`, select `Anyone, even anonymous` on the `Who has access to the app` menu, and click `Deploy` button
1. Select the `Current web app URL`. This is the endpoint URL for receiving event logs from Fluentd. Copy and paste the URL to clipboard or anywhere to use it later
1. Select `Run` - `doPost` menu, click `Continue` button of the `Authorization Required` dialog and click `Accept` button on the `Request for Permission` dialog
1. Confirm that there are a line chart appeared on the spreadsheet. Now it's ready to accept event logs from Fluentd

You can choose one from the two options to use the spreadsheet.

### Option A: Use with `fluent-plugin-https-json`:

In this option, you can use the spreadsheet as a dashboard for any event log collected by Fluentd. This option is useful when you don't require any complex analytics and just want to show Fluentd logs on the spreadsheet.

1. [Configure a host for Fluentd installation](https://www.google.com/url?q=http://docs.fluentd.org/articles/before-install&usd=2&usg=ALhdy2-Eq3wSUPNxaZr13oC2Mt5UssbUhw)
2. Install [fluent-plugin-https-json](https://github.com/jdoconnor/fluentd_https_out)
4. Edit `td-agent.conf` to add the following match element. Replace the `<<ENDPOINT URL>>` with your endpoint URL and edit the `**` pattern if needed. Save the file and restart td-agent

```
    <match **>
        type            https_json
        use_https       true
        buffer_path     /tmp/buffer
        buffer_chunk_limit 256m
        buffer_queue_limit 128
        flush_interval  3s
        endpoint        <<ENDPOINT URL>>
    </match>
```
td-agent.conf

### Option B: Use with `fluentd-norikra-gas` Docker image:

In this option, you can use the spreadsheet as a dashboard for [Norikra](http://norikra.github.io/), a Complex Event Processing (CEP) tool that let you use SQL for fast and scalable event log aggregation.

1. [Configure a host for Fluentd installation](https://www.google.com/url?q=http://docs.fluentd.org/articles/before-install&usd=2&usg=ALhdy2-Eq3wSUPNxaZr13oC2Mt5UssbUhw)
2. [Prepare a Docker environment](https://www.google.com/url?q=https://www.docker.io/&usd=2&usg=ALhdy2-uNZKLM-jQQXncnc5eKHG-11c4og)
3. Execute the following docker command with putting the endpoint URL at the place of `<<ENDPOINT URL>>`

```
$ sudo docker run -p 26578:26578 -p 26571:26571 -p 24224:24224 -p 24224:24224/udp -e GAS_URL=<<ENDPOINT URL>> -t -i -d kazunori279/fluentd-norikra-gas
```

Now the host works as a Fluentd + Norikra server. Configure your Fluentd clients to forward logs to the host, and add Norikra queries by using its Web UI. The query result will be displayed as a new sheet on this spreadsheet. See [this site](http://norikra.github.io/) for details of Norikra.

### Notes:

- The spreadsheet can only receive one event log per a few seconds for each sheet. You can use it for receiving aggregated statistics (such as req/s, average CPU/memory usage etc) every few seconds, rather than using it for receiving raw streaming events. Recommended event rate is 1 event per 3 seconds for each sheet.
- When the spreadsheet receive an event log with a new tag name, it creates a new sheet with the Fluentd tag name (or Norikra query name)
- If the tag name has a suffix `_AREA`, `_BAR`, `_COLUMN`, `_LINE`, `_SCATTER`, or `_TABLE`, it will also create a new sheet with a specified chart
- If the tag name has a suffix `_AREA_STACKED`, `_BAR_STACKED` or `_COLUMN_STACKED`, it will create a stacked chart
- The endpoint URL does not support authentication. Please make sure to keep the URL secret and not to make it public

## Use with BigQuery for historical analytics

To use the spreadsheet with BigQuery for historical analytics, follow the instructions below for enabling BigQuery access from the spreadsheet. If this is the first time to use BigQuery, please refer to the [BigQuery Getting Started page](https://developers.google.com/bigquery/sign-up).

1. Select `Tools` - `Script editor...` menu
1. On the Script editor, open `bq_query.gs` and paste Project ID of your Google Cloud Platform project on the place of `<<PLEASE PUT YOUR PROJECT ID HERE>>`. Select `File` - `Save` menu to save the file
1. Select `Resources` - `Advanced Google services` menu and turn on `BigQuery API`
1. Click `Google Developers Console` link of the dialog. This will show a list of APIs. Find `BigQuery API` and click `OFF` button to enable it
1. Close the Console, click OK button of the dialog

### Execute a sample query:

Now it's ready to execute BigQuery queries from the spreadsheet. Use the following instructions to try executing a sample query.

1. Open the spreadsheet and open `BQ Queries` sheet. The sheet has a sample BQ query named `gsod_temparature_LINE` which aggregates temparature data of each year from the public GSOD dataset available on BigQuery
1. Select `Dashboard` - `Run All BQ Queries` menu. For the first time, it will show a dialog `Authorization Required`. Click `Continue` button and then `Accept` button
1. There will be a `gsod_template` sheet added. Open the sheet and check there are the query results
1. Open the `Lambda Dashboard` sheet and check there is a graph added for the query results

### Automatic query execution:

If you like to execute the queries periodically, use the following instructions.

1. Open the Script editor and select `Resources` - `Current project's triggers`
1. Click `Click here to add one now`
1. Select `runQueries` for `Run` menu, and select `Time-driven` `Minutes timer` `Every minute` for `Events`, and click `Save`
1. Go back to `BQ Queries` sheet, set `1` to the `interval (min)` column of the `gsod_temperature_LINE` query

By this setting, the sample query will be executed every one minute. Set `0` to the `interval (min)` to disable the periodical execution.


### Notes:

- When the spreadsheet executes a query with a new query name, it creates a new sheet with the query name
- If the tag name has a suffix `_AREA`, `_BAR`, `_COLUMN`, `_LINE`, `_SCATTER`, or `_TABLE`, it will also create a new sheet with a specified chart
- If the tag name has a suffix `_AREA_STACKED`, `_BAR_STACKED` or `_COLUMN_STACKED`, it will create a stacked chart
- Put `LIMIT 100` at the end of each query to limit the lines of query result to 100. Otherwise it may throw an error when the results exceed the limit
- The first field of the query results should be timestamp or date value to draw a chart with chronological order



