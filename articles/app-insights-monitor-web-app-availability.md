<properties title="Monitor any web site's availability and responsiveness" pageTitle="Monitor any web site's availability and responsiveness" description="Set up web tests in Application Insights. Get alerts if a website becomes unavailable or responds slowly." metaKeywords="analytics web test availability" authors="awills"  manager="kamrani" />

<tags ms.service="application-insights" ms.workload="tbd" ms.tgt_pltfrm="ibiza" ms.devlang="na" ms.topic="article" ms.date="2014-12-11" ms.author="awills" />
 
# Monitor any web site's availability and responsiveness

After you've deployed your web application, you can set up web tests to monitor its availability and responsiveness. Application Insights will send web requests at regular intervals from points around the world, and can alert you if your application responds slowly or not at all.

![Web test example](./media/appinsights/appinsights-10webtestresult.png)

You can set up web tests for any HTTP endpoint that is accessible from the public internet.

*Is it an Azure website? Just [create the web test in the website blade][azurewebtest].*


1. [Create a new resource?](#create)
1. [Set up a web test](#setup)
1. [View results](#monitor)
2. [If you see failures...](#failures)
2. [Multi step web tests](#multistep)
1. [Edit or disable a test](#edit)


 [Video](#video)
 [Next steps](#next)

## Setting up a web test

### <a name="create"></a>1. Create a new resource?

Skip this step if you've already [set up an Application Insights resource][start] for this application, and you want to see the availability data in the same place.

Sign up to [Microsoft Azure](http://azure.com), go to the [Preview portal](https://portal.azure.com), and create a new Application Insights resource. 

![New > Application Insights](./media/appinsights/appinsights-11newApp.png)

### <a name="setup"></a>2. Create a web test

In the overview blade for your application, click through the webtests tile. 

![Click the empty availability test](./media/appinsights/appinsights-12avail.png)

*Already got some web tests? Click through the webtests tile, and choose Add Webtest.*

Set up the test details.

![Fill at least the URL of your website](./media/appinsights/appinsights-13availChoices.png)

- **The URL** must be visible from the public internet. It can include a query string - so for example you could exercise your database a little. If the URL resolves to a redirect, we will follow it, up to 10 redirects.

- **Test locations** are the places from where our servers send web requests to your URL. Choose two or three so that you can distinguish problems in your website from network issues. You can't select more than three.

- **Success criteria**:
    **HTTP return codes**: 200 is usual. 

    **Content match** string, like "Welcome!" We'll test that it occurs in every response. It must be a plain string, without wildcards. Don't forget that if your page content changes you might have to update it.

- **Alerts**: By default, alerts are sent to you if there are repeated failures over 15 minutes. But you can change it to be more or less sensitive, and you can also change the notified email addresses.

#### Test more URLs

You can add more tests for as many URLs as you like. For example, as well as testing your home page, you could make sure your database is running by testing the URL for a search.

![On the web tests blade, choose Add](./media/appinsights/appinsights-16anotherWebtest.png)


### <a name="monitor"></a>3. View availability reports

After 1-2 minutes, click Refresh on the overview blade. (In this release, it doesn't refresh automatically.)

![Summary results on the home blade](./media/appinsights/appinsights-14availSummary.png)

The chart on the overview blade combines results for all the web tests of this application.

#### Page components

Images, style sheets and scripts and other static components are requested as part of the test.  

The recorded response time is the time taken for all the components to complete loading.

If any component fails to load, the test is marked failed.

## <a name="failures"></a>If you see failures...

Click through to the webtests blade to see separate results for each test.

Open a specific web test.

![Click a specific webtest](./media/appinsights/appinsights-15webTestList.png)

Scroll down to **Failed tests** and pick a result.

![Click a specific webtest](./media/appinsights/appinsights-17-availViewDetails.png)

The result shows the reason for failure.

![Webtest run result](./media/appinsights/appinsights-18-availDetails.png)

For more detail, download the result file and inspect it in Visual Studio.



##<a name="multistep"></a>Multi-step web tests

You can monitor a scenario that involves a sequence of URLs. For example, if you are monitoring a sales website, you could test that adding items to the shopping cart works correctly. 

To create a multi-step test, you record the scenario by using Visual Studio, and then upload the recording to Application Insights. Application Insights will replay the scenario at intervals and verify the responses.

#### 1. Record a scenario

Use Visual Studio Ultimate to record a web session.

1. Create a web performance test project.

    ![In Visual Studio, create a new project from the Web Performance and Load Test template.](./media/appinsights/appinsights-71webtest-multi-vs-create.png)

2. Open the .webtest file and start recording.

    ![Open the .webtest file and click Record.](./media/appinsights/appinsights-71webtest-multi-vs-start.png)

3. Do the user actions you want to simulate in your test: open your website, add a product to the cart, and so on. Then stop your test. 

    ![The web test recorder runs in Internet Explorer.](./media/appinsights/appinsights-71webtest-multi-vs-record.png)

    Don't make a long scenario. There's a limit of 100 steps and 2 minutes.

4. Run the test in Visual Studio to make sure it works.

    The web test runner opens a web browser and repeats the actions you recorded. Make sure it works as you expected. 

    ![In Visual Studio, open the .webtest file and click Run.](./media/appinsights/appinsights-71webtest-multi-vs-run.png)
 

(Don't insert loops in your web test code.)

#### 2. Upload the web test to Application Insights

1. In the Application Insights portal, create a new web test.

    ![On the web tests blade, choose Add.](./media/appinsights/appinsights-16anotherWebtest.png)

2. Select multi-step test, and upload the .webtest file.

    ![Select multi-step webtest.](./media/appinsights/appinsights-71webtestUpload.png)

View your test results and any failures in the same way as for single-url tests. 

A common reason for failure is that the test runs too long. It mustn't run longer than two minutes.


### Plugging time and random numbers into your multi-step test

Suppose you’re testing a tool that gets time-dependent data such as stocks from an external feed. When you record your web test, you have to use specific times, but you set them as parameters of the test, StartTime and EndTime.

![A web test with parameters.](./media/appinsights/appinsights-72webtest-parameters.png)

When you run the test, you’d like EndTime always to be the present time, and StartTime should be 15 minutes ago.

Web Test Plug-ins provide the way to do this.

1. Add a web test plug-in for each variable parameter value you want. In the web test toolbar, choose **Add Web Test Plugin**.

    ![Choose Add Web Test Plugin and select a type.](./media/appinsights/appinsights-72webtest-plugins.png)

    In this example, we’ll use two instances of the Date Time Plug-in. One instance is for "15 minutes ago" and another for "now". 

2. Open the properties of each plug-in. Give it a name and set it to use the current time. For one of them, set Add Minutes = -15.

    ![Set name, Use Current Time, and Add Minutes.](./media/appinsights/appinsights-72webtest-plugin-parameters.png)

3. In the web test parameters, use {{plug-in name}} to reference a plug-in name.

    ![In the test parameter, use {{plug-in name}}.](./media/appinsights/appinsights-72webtest-plugin-name.png)

Now upload your test to the portal. It will use the dynamic values on every run of the test.

## <a name="edit"></a> Edit or disable a test

Open an individual test to edit or disable it.

![Edit or disable a web test](./media/appinsights/appinsights-19-availEdit.png)

You might want to disable web tests while you are performing maintenance on your service.

## <a name="video"></a>Video

> [AZURE.VIDEO monitoring-availability-with-application-insights]

## <a name="next"></a>Next steps

[Search diagnostic logs][diagnostic]

[Troubleshooting][qna]




[AZURE.INCLUDE [app-insights-learn-more](../includes/app-insights-learn-more.md)]




[azurewebtest]: ../insights-create-web-tests/