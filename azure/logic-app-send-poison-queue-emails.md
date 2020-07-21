# Use Logic App to Send Poison Queue Emails

You can setup an Azure Logic App that will montior all your *-poison queue names. Once a message was found you can send an email with the queue message contents.

Setup:
1. Create a Logic App. Best to include the expected outcome in the app name.
2. In the designer setup the `Recurrence` step. I have it running once an hour to save costs.
3. Create a `List queues` step.
4. Create a `For each` step that will loop thru the queue `Body`.
5. Create an `And` condition with queue `Name` `ends with` `-poison`.
6. For the `If true` condition add a `Get messages` action. Set Queue Name as queue `Name`. Set the number of messages to `20` any more can be beneficial but will feel like spam.
7. Add another `For each` action in the `If true` that will loop thru the `QueueMessage` and send an email with whichever email provider is setup. 

Subject: Poison Queue Messages - `Name`
Body: `Message Text`
8. After sending the email you will want to delete the queue message so it doesn't keep re-sending an email. Add an action to `Delete message`. Queue Name: `Name`, Message ID `Message ID` and Pop Receipt `Pop Receip`.


Feel free to test this out by adding a message onto any `*-poison` queue. Go back to Azure Portal for your Logic app and `Run Trigger`.