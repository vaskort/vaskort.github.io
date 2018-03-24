---
layout: post
title: Azure Webjobs - Deploy and manage them from your Node App
---

Have you ever wanted to manage your Webjobs directly from your Node app instead of having to go to the Azure portal?
Besides writing the code for them you can define if they would be scheduled or manual and also set the CRON expression for them.

There is a specific folder structure you should follow so that Azure can list your js files in the Webjobs and its the below:

```
App_Data 
│
└──jobs
   │
   └───triggered
       │   
       └───demo-webjob
            │
            └─run.js
```

Create your `run.js` file in the above structure and push your changes. Refresh your Webjobs in Azure and you should be able to see it.

![Webjob]({{ "/images/post-2/webjob.png" | absolute_url }})

Sweet. We now have a triggered webjob but we haven't scheduled any CRON expression for it yet. We just need to add a `settings.job` file in our webjob folder

```javascript
//settings.job
{
  "schedule": "0 */15 * * * *"
}
```

The six fields of a CRON expression are:
```
{second} {minute} {hour} {day} {month} {day-of-week}
```

The above CRON expression will run our webjob every 15 minutes. Check [this link](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer#cron-examples) for some CRON examples.

After adding the `settings.job` you should be able to see your CRON expression appear under the `SCHEDULE` column.

![Webjob-CRON]({{ "/images/post-2/cron.png" | absolute_url }})