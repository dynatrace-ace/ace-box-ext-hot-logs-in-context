## Lab 4: Security Use Case
 
### Preparation/Context 
 
1.	In your Dynatrace environment, navigate to the **Workflows app**. 
2.	Open and run the **Security Lab Workflow**. 
 
This workflow empties the *EasyTrade* accounts for a few select customers.   

In this lab exercise, we will analyze logs by using the *Security Investigator* app to identify which user accounts are impacted, who emptied the accounts, and where the money went. 

1.	To start this exercise, we will use a new app called the **Security Investigator**. As we did before, simply use the search function to find and open this app  

![Security Investor](../../assets/images/SecurityInvestigator.png)

2.	Once you have opened the app, create a new case and rename it **Account Theft**  

![Account Theft Case](../../assets/images/AccountTheftCase.png)

3.	We will be reusing the same query from the previous lab exercise as a starting point for our investigation. Replace the existing DQL query with **the following query**: 

```DQL
fetch logs 
| filter k8s.namespace.name == "easytrade" 
| filter contains(content, "balance", caseSensitive:false) 
| fields timestamp, content, accountId, actionType, balance, oldValue, valueChange 
| sort timestamp desc
```

4.	Click the **Play button** to run the query. 
  
![Play Button Location](../../assets/images/PlayButton.png)

This provides us with relevant log records in the selected timeframe. These results (thanks to the fields portion of the command) limit the resulting fields shown to those we are interested in. 
 
Notice that after running our query, the **query tree** on the top right side, now includes a *node – Query#1*. We will further explore how to manage these later. 

Additionally, note the **evidence collection** section on the bottom right side. This allows us to track interesting data as we come across it during our investigation. We will make use of this later as well. 

5.	Since we know that money has been taken out of accounts, we want to investigate log records related to *withdrawal transactions*. Scroll to the right until actionType column is visible. Look through records for one with the actionType value **withdraw**. Right click on the field and select “**Filter for**” and run the query.
 
![Filter for withdraw](../../assets/images/FilterForWithdraw.png)

This will filter out the data for all the withdrawals. Notice the addition of the last filter command and the second node in the query tree. 

6.	We’ve been notified that several accounts have been emptied – this is where we want to focus our attention. Let’s identify accounts that fit this criterion. *How can this be achieved with the information available?*

Each record has values for *oldValue* and *valueChange* 

We can calculate the remaining balance for each transaction by adding the following command to the end of our query: 

```DQL
| fieldsAdd remainingBalance = oldValue + valueChange 
```

7.	Re-run **the query**. When looking at the results, we see entries with a *remainingBalance of 0* – let’s filter on these by right-clicking the cell and selecting **Equal** 

![Equals Filter](../../assets/images/Equal0.png)

8.	Re-run **the query**. We now see suspicious transactions matching the criteria given before. Let’s keep track of this by renaming the corresponding node in the query tree to **Suspicious transactions**. Right-click the node and select **Rename** 
 
![Renaming as Suspicious Transactions](../../assets/images/RenameSuspicousTransaction_new.png)

We can additionally change the color of the node to highlight it as an important part or finding in our investigation process: 

![Change Color of Important Node](../../assets/images/ChangeColorNode_new.png)

9.	We must do further investigation to understand who withdrew the money and understand where it went, for each of these accounts that we discovered. There are several ways to approach this.  
 
Let’s modify our query to look at all log records that contains one of the IDs using the following query for the **accountId** `62` that was one of the suspicious accounts:  

```DQL
fetch logs  
| filter k8s.namespace.name == "easytrade"  
| filter accountId == 62 
| sort timestamp desc  
```


Run the query. The results show records relating to the specific account. Right-click a record starting with `info: easyTradeBrokerService.Controllers.CreditCardController`and select View field details to expand the content field.  
 

The resulting modal shows us several pieces of information that are of interest: 
- *AccountId*
- *Amount*
- *CardNumber* 
- *Email*

![Withdrawal Records](../../assets/images/WithdrawalRecords.png)

With this information, it is possible to compare to **Updating account** with data on the row just above and see that the names are not correlating, the person who withdraw money is not the same as the account holder. 

10.	To keep the query tree clean (so we or others can understand our steps), rename the corresponding node for this query to **Identify withdrawal information**. Additionally, changing **the color** of the node can be useful (visually) to show that this query is not directly tied to the specific transactions we want to investigate, but it is still a step in the right direction.
 
![Identify Withdrawal Information Color Change](../../assets/images/ChangeColorIdentifyWithdrawalInformation.png)

The resulting log records provide us with important details that explain what happened, including:  

- When the balance history was updated  
- Who withdrew the money  
- Who the impacted account belongs to  

Time to extract these pieces of valuable information and add them to the evidence list!   

11.	Create new evidence collections to store our findings. We will create 3 collections:  

- Credit Card Number  
- Account ID  
- Impacted User   

To create a new evidence collection, click the **+ button**. Add the corresponding title. Ensure the evidence type is String, then click **Confirm**. 

![Evidence Collection](../../assets/images/EvidenceCollection.png)

![New Evidence](../../assets/images/NewEvidence.png)

With evidence collections created, we can now extract these values for the first suspicious transaction by looking at the results we generated previously. (step 9) 

12.	Right-click the bottom log record containing “**Withdrawing money with data**”, then select **View field details** 
  
![Adding Balance History with Data Details](../../assets/images/withdrawingmoneywithdata.png)

- Add the **AccountId** value to the Account ID evidence collection: 
- Highlight the **account ID value**
- Right-click and hover over **Custom lists** 
- Select **Account ID** 
 
![Account ID](../../assets/images/AccountID.png)

13.	Add the **CardNumber value** to the **Credit Card Number** evidence collection: 
    - Highlight the **CardNumber value** 
    - Right-click and hover over **Custom lists** 
    - Select **Credit Card Number** 

![Card Number](../../assets/images/CardNumber.png)
  
14.	Click the **left arrow** in the modal again to view the **previous log record**. 
    - Add the **email address** of the user affected to the **Impacted Users** evidence collection: 
    - Highlight the **Email value** 
    - Right-click and hover over **Custom lists** 
    - Select **Impacted Users** 

![Impacted Users](../../assets/images/ImpactedUsers.png)
  
All values should now be seen in the Evidence collection section: 

![Evidence Collection Section](../../assets/images/EvidenceCollectionSection.png)
 
 
**This completes our investigation for this account!** Evidence for the other affected accounts can be collected by repeating these steps. We can contact the affected users to notify them of the suspicious activity and our findings. 
