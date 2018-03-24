---
layout: post
title: Azure Webjobs - Deploy and manage them from your Node App
---

Have you ever wanted to manage your Webjobs directly from your Node app without having to browse the Azure portal?  
You can also define if they would be **triggered** or **continuous** and also set the CRON expression for the triggered one.

### Folder structure for your webjobs
There is a specific folder structure you should follow so that Azure can pick up your Javascript files and show them in the Webjobs settings.

{% highlight bash %}
App_Data 
│
└──jobs
   │
   └───triggered
       │   
       └───demo-webjob
            │
            └─run.js
{% endhighlight %}

Create your `run.js` file in the above structure and push your changes. Refresh your Webjobs in Azure portal and you should be able to see it. If not, refresh again. :grimacing:

![Webjob]({{ "/images/post-2/webjob.png" | absolute_url }})

### Add the CRON expression
Sweet. We now have a triggered webjob but we haven't scheduled any CRON expression for it yet. We just need to add a `settings.job` file in our webjob folder.

{% highlight javascript %}
//settings.job
{
  "schedule": "0 */15 * * * *"
}
{% endhighlight %}

The six fields of a CRON expression are:
{% highlight javascript %}
{second} {minute} {hour} {day} {month} {day-of-week}
{% endhighlight %}

The above CRON expression will run the webjob every 15 minutes. Check [this link](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer#cron-examples) for some CRON examples.

After adding the `settings.job` you should be able to see your CRON expression appear under the `SCHEDULE` column.

![Webjob-CRON]({{ "/images/post-2/cron.png" | absolute_url }})

### Continuous Webjob

Great, we have added a triggered webjob and set up how often will it run. You can follow the same process to add a continuous webjob. The difference between the two is that the continuous one will run only once its created. If you want it to run endlessy you have to handle that inside your code.  

So the full folder structure for our two webjobs would be:

{% highlight bash %}
App_Data 
│
└──jobs
   │
   |───triggered
   |    │   
   |    └───demo-webjob
   |         │
   |         └─run.js
   └───continuous
        |
        └───continuous-webjob
            |
            └─anotherWebjob.js
{% endhighlight %}
And the final result in Azure portal would be:  

![Webjob-2]({{ "/images/post-2/webjob-2.png" | absolute_url }})

### Notes - Links

Folder structure on GitHub: [https://github.com/vaskort/vaskort.github.io/tree/second-post/App_Data](https://github.com/vaskort/vaskort.github.io/tree/second-post/App_Data) 

Azure Webjobs documentation: [https://docs.microsoft.com/en-us/azure/app-service/web-sites-create-web-jobs](https://docs.microsoft.com/en-us/azure/app-service/web-sites-create-web-jobs)  

Cron examples: [https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer#cron-examples](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer#cron-examples)  