## Lab 1: Getting Started with Logs
 
### What will this lab focus on?
This is an entry-level lab to get started with the basics of Logs in Dynatrace. In this lab we will get started with navigating in the Dynatrace UI, use the Problems App and the Logs app to create basic log queries.

### Lab Objective
Participants will be first able to navigate within Dynatrace UI, identifying a problem in a simple and easy way. Then they will be able to drill down and understand:  
- the **overall impact** of the problem  
- what’s the exact **root cause**   
- pinpoint the **logs** that are strictly related to that issue.   
- go through further analysis exploring the logs, checking and querying the **logs in the same context of the problem**. It’s important to show how we can analyze logs content, and get value out of logs (**WITHOUT DQL**, and then drill down using *DQL* (within *Notebooks* and/or *Dashboards*))

### Lab Scenario
A problem has been generated that makes it very difficult for users to login to the Unguard app. We will use Dynatrace to analyze this problem.

### Why is knowing/doing this important? 
The Logs App allows you to easily find relevant logs for your applications and can serve as a basis of further analysis. It also allows you to query your logs without using DQL (Dynatrace Query Language).

### Step 1: Finding and viewing a problem
1. Access your **Dynatrace tenant** provided for the lab
2. Open the “**Problems**” app by clicking on it from the sidebar or by using the search function available at the top left.

![Problems](../../assets/images/01_open_problem.png)

3. Within the problems app screen you will be able to see a list of **open**/**closed** **problems**
4. For the lab we will select a particular problem of “**Failure rate increase**”. Click on the **problem ID** to open the details of that problem.  

![Failure Rate Problem Card](../../assets/images/01_problem.png)

5. Click on **properties** to view additional metadata for the selected entity

### Step 2:  Selecting and analyzing relevant logs related to the problem

There are multiple apps at your disposal for viewing/analyzing logs. For initial analysis we will use the **Logs app**. This app provides an *UI based approach* without the need for DQL or any sort of querying knowledge.

1. Use the **search function** at the top left of the sidebar and search for the “**Logs**” app  

![Logs App](../../assets/images/01_open_logs.png)

2. With logs app open, set the timeframe to “**last 2 hours**” to match our problem timeframe
3. Click on **Run Query** to view all the logs for the last 2 hours

We can now see *all* the logs that were generated in the last two hours. But we want to narrow it down to investigate our problem and show only relevant logs. 

4. To filter the result, click in the filter field and select **status**. Then select **=** and “**ERROR**” from the preview. This will filter logs with errors.
5. 
![Filter on Status](../../assets/images/01_filterForError.png)

6.	Further, use the content box to filter for all logs related to **login**. You can do this simply selecting content in the filterfield, select '=' and then **\*value\***. Set this to **"\*login\*"**

![Filter on Content](../../assets/images/01_filterForLogin.png)

7. Now click on **Run Query** for these filters to be applied. *Remember*, the filtering options will not be applied automatically. After every change you have to click “*run query*” for the changes to be applied.

Now we can see all the error logs related to login for the selected timeframe, but from the problem card we already know that the issue is with the **user-auth-service**.

By default, the logs app shows a few basic filters and most common columns, but you can add a column for any of the available *properties* that you desire and add a filter for it.

8. Click on “**xx columns hidden**” to reveal all available columns. Select "**k8s.namespace.name**" and “**k8s.container.name**”. Click on **confirm** to add them to the analysis view

![Select these columns](../../assets/images/01_columnNamesVisible.png)
 
9. Now click on **any log line** to view the log contents and all the metadata for it. In this screen you can use any of the metadata to filter the result. Look for *k8s.container.name* and click on the **filter icon** next to it and add it as filter.  

![Select these columns](../../assets/images/01_filterForContainer.png)

10. Finally, let's also filter for this namespace. Instead of opening the log record, we can also click on the three vertical dots in a record and filter on those values. Click on the three dots next to unguard and add this as a filter.

![Select these columns](../../assets/images/01_filterForNamespace)

So far, we've been interacting with the UI in a really simple way without the need for extensive querying knowledge. What's behind each of these queries? It is written using *Dynatrace Query Language*, also known as *DQL*. 

We have narrowed down on the problem and filtered the relevant logs, but how can we perform further analysis on this data, or even store or share this data with other members or teams. We can simply do this by exporting this result into another app called "**Notebooks**".

### Step 3: Storing, visualizing and sharing log analysis
1. Click on the “**open with**” button to use this data with another app within Dynatrace. In the subsequent screen select “**Notebooks**”

![Open With](../../assets/images/01_clickOpenWith.png)
![Open With](../../assets/images/01_selectNotebook.png)
![Open With](../../assets/images/01_new_notebook.png)

This opens our log query in a 'Logs' type section in a Notebook. Notebooks allow us to store queries and perform deeper analyses than just viewing the log contents.

We can then turn the query result into a *chart* with a really simple command that will allow us to **count** *occurrences* **by** *status*. In this way we will be able to show (for the defined timeframe) the *number of log occurrences* fulfilling these conditions, *grouped by status*, showing the overall number of rows with *5 minutes intervals* for example. 
 
2. To achieve this result we need to turn our result into timeseries. We can do that by clicking "**+ Command**", selecting **Convert to Timeseries** and running the query. We also have to adjust the visualisation.

![Convert to Timeseries](../../assets/images/01_convertToTimeseries.png)
3. To render this as a line chart, click **Options**, set **Line** in visualisation, and set the dots to connected in the **Lines** part.
![Select Line Chart](../../assets/images/01_selectLineChart.png)
![Connect Dots](../../assets/images/01_setConnectDots.png)

This shows us the number of failed logins every minute.

We can analyze deeper to create more insightful reports and visualizations. *Notebooks* are a powerful tool. It is out of scope for this lab to explore all the possibilities, but we will take a sneak peak of what is possible. We are going to extract *all* the affected **user(names)** for this login issue. 
 
Now we can extract the different users for status errors, showing who faced the issue.

4. To achieve this we need to *parse* the data and format the result using *DPL*. You can simply use **the below query** and paste it in your notebook. First, create a new section and select **DQL** as the type. Then copy the query, paste it and run it.

```DQL
fetch logs 
| filter status == "ERROR" 
| filter contains(content, "login") 
| filter k8s.container.name == "user-auth-service" 
| filter k8s.namespace.name == "unguard" 
| parse content, """DATA //  
DATA 'username\":\"' LD:username '\"'""" 
| summarize countDistinct(username), by:{username, status} 
| fieldsRemove `countDistinct(username)` 
```
![User Extraction](../../assets/images/UserExtraction.png)

**You have successfully completed lab 1**.  As discussed we saw how easily and quickly we went from a problem alert to the exact root cause while also gaining business insights. 

