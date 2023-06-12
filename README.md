# [Charles Proxy](https://www.charlesproxy.com/overview/about-charles/) Debugging Tool for Mobile App Testing

<img src="https://user-images.githubusercontent.com/70295997/226082568-0a72d438-99c3-4962-8e16-724df840d581.png" width=600>

Almost every software product, whether it’s web or mobile, has some kind of tracking implemented. Tracking is used for gathering insights on how the product is used. User flows can be tracked to improve the way users are working with the product. However, while user tracking can be useful it can also be misused.

If a mobile app is using tracking, the mobile team should ask themselves if it's really necessary to track and if it's helpful for the business? There are laws in many countries that protect user data and how it’s used. The EU General Data Protection Regulation (GDPR) is the law focusing to consolidate the data privacy laws across the EU. Companies have to inform their users about the implemented tracking and why and how the data is used. If a company does not follow the law, it can be sued and fines can be assessed for violation of the law. Whenever tracking is implemented, mobile teams should know the regulations for user tracking and data, so they do not open their company to liability inadvertently.

If tracking is in place, make sure it works before every release. Use tools like Charles Proxy or Fiddler to verify. Charles Proxy is what I've been using as a tester. When conducting Mobile App Testing, I have Charles running on my machine 3/4 of the time.

For *Android* versions of the app *only*, the developers provide testers with a special debuggable build that we can use with Charles Proxy. They [add a Network Security Configuration file into the Android project](https://medium.com/@ferrygunawan/debug-ssl-traffic-with-charles-proxy-872aedb228d#be19) in order for us to be able to read the traffic in Charles. The production versions do not have this file. We only have it in the test versions, the debug builds, and only if the developers have added the configuration file. 99% of the time the test version of the app is already configured for use with the Android version. iOS does not require that, only Android does. Without the special debug build, we aren't able to view the traffic.

Overall, the typical debugging activities I conduct, as a mobile tester, with Charles Proxy are:

1. [Logging](https://github.com/lana-20/charles-proxy/blob/main/README.md#logging)
2. [Throttling](https://github.com/lana-20/charles-proxy/blob/main/README.md#throttling)
3. [Map Local](https://github.com/lana-20/charles-proxy/blob/main/README.md#map-local)
4. [Re-Write](https://github.com/lana-20/charles-proxy/blob/main/README.md#re-write)
5. [Breakpoints](https://github.com/lana-20/charles-proxy/blob/main/README.md#breakpoints)

## Logging

The **Activity Logging** feature of a proxy server is instrumental in debugging issues during the root cause analysis. It helps us pinpoint a failure to either the front- or the back-end. Mobile Testing often involves utilization of proxies to read the traffic between the app's front-end & back-end (and any nook & cranny, ebbs and flows that a request or response may traverse through). For extensive logging and root cause analysis, I use Charles Proxy. 

The request or response can encounter issues in many different places, while travelling over HTTP from the client to the server and back. What are those places?

Karl von Randow is an experienced developer who started over 20 years ago when Chrome or Safari did not exist yet. There were no good debugging tools. Nowadays, we can check for any issues with our Requests in the browser DevTools (for a web-based app). We can inspect the page, analyze multiple tabs such as Elements, Console, Sources, Network, Performance, Memory. We can go to our Network Tab and figure out which end-point we was trying to hit, our status code, how much time it took, etc. We can check it in the DevTools, but Karl couldn’t. At that time, there was no such thing as Chrome DevTools, there was no Chrome. He created the ‘man in the middle’ who’d help him to debug his web-based app. For example, to figure out why he did not see things that he would expect to see, and what was failing. Not having a debugging tool may cause a lot of frustration. So, he created Charles Proxy.

Karl posed the following 7 questions:

1. **Did I send a request?**
2. **Was the request correct?**
3. **Did the server fail?**
4. **Did the network fail?**
5. Did I receive a response?
6. Was the response correct?
7. **Did the client fail to handle the response?**

One may ask, "Why not use Postman?". Postman can partially address these questions.

With Postman, we can only answer questions 5 and 6. Did I receive the response, and was it correct? 

Postman is a great tool, helpful in situations such as:
- When we have the back-end ready, but the front-end UI is not done yet. We can test features early, check if the API works well, before it gets connected to the UI.
- Create a Collection to help us generate data. Let's say, to run Regression Testing, we need an account that has multiple reservations and multiple places. Creating this data manually is tedious. We can create Postman Collections that generate the data for testing. A Collection can create a few places and make a few reservations for us.

But we don't exclusively work with the back-end, when testing a mobile or a web-based app. I do use Postman, but not as often as one might think. Once in a while, I might run some Collections or run the requests one by one.

Postman is not the ‘man in the middle’. It has no idea what’s going on on the front-end. It's just a tool that lets us, as testers, play the role of the app and request data from the server on behalf of the app. But if we want to know more and get answers to all of the 7 questions, we're looking into using a secure proxy server to intercept (listen in on) the traffic.

Privacy, speed, bandwidth savings, activity logging are the benefits of using a Proxy Server. Charles Proxy takes advantage of one of the abilities of proxy servers. This ability is activity logging. Charles sticks in the middle, records communication activities between client and server -- and we can view it in real time, it helps us identify and isolate the issue.

When can such activity **logging** be useful? Let's say, you **open an app and see less content that you expect**. How would you approach that?

For example, I am working on an app that reserves public transportation for visually impaired users. In this app, I can save various places to *My Places*. For testing purposed. I have 3 places saved already. But when I navigate to *My Places* screen, I only see 2. There's an obvious issue here. How to approach it? I could check the database and get a result set of 2. That would mean a UI issue, and I'd file a bug report with my mobile developers not the back-end ones.

However, things can go wrong in many places, there is a long way between the UI and the database. I could query the database and get 3 places in return. But that would not immediately indicate a UI bug. In order to be sure, I need to closely trace how my app's UI side communicates with the backend. Apparently, when I login I do not query the database directly. There are all sorts of API methods - GET, PUT, POST, PATCH, DELETE. When I look for *My Places*, the app makes an API request to GET places for the user. I know which users are logged in when I'm in my app. For a particular logged in user, the app asks the back-end to provide their places. Then the back-end goes and retrieves those places from the database. Behind the scenes, it means that once the app executes this request, asking for places for a specific user, the back-end selects places from the Places table:

    SELECT place from Places
    WHERE user_id = ' ... '

There is a chance that a developer made a mistake when composing a query. They may forget something or not GROUP by something. We might have to query multiple tables. The developers are human, they can make mistakes.

When the 'GET places for the user' request does not return all of *My Places*, I instantly go into the database directly. If I write a correct query, but my the devs didn't, I retrieve the correct result set of 3 places. But there is no error thrown to indicate an issue. I should have started debugging by first figurng out what the app was actually requesting.

Did the app actually request the places? What if it requested something else? What if there are problems with the API call? What if the app actually executes POST or DELETE in lieu of GET? What if there's a problem with the what tha app is asking for? What if the developers wrote a query with some issues, which returns an inaccurate result set?

A lot of things can happen. It can be hard to figure out if the issue belongs to the front- or back-end. We may encounter issues in many places, highlighted in the **7 questions** posed by the creator of Charles.

1. **Did I send a request?**
    - I may not be sending any requests. When debugging such a situation, in the app we want to check if it even sent the request. Did the app even request anything again? Perhaps, I am seeing 3 places because they are coming from the cache, and the app does not make another request every time I open *My Places* screenview. Did I even send the request?
2. **Was the request correct?**
    - If I sent the request, was this request correct? Did I actually request places for the particular user or something else? Or, did I somehow get a hard coded used_id who is not my currect user?
3. **Did the server fail?**
    - If the request was correct, did the server fail? Or, did the network fail? We use the internet connection, and there may be some interruptions along the way; the network can fail. When my app communicates with the back-end, mostly I use the HTTPS protocol. HTTPS uses the network, otherwise it wouldn't work. So, it may be a network failure.
4. **Did the network fail?**
5. **Did I receive a response?**
    - Did I receive any response at all? Let's say, I request *My Places* for the app user, but nothing comes back, the server sends no responsem, and I still see the 2 places that have been cached from the last time. If I go to the database directly and still get the result set with 3 places, it is still unclear where the problem lies.
6. **Was the response correct?**
    - If I do receive a response from the server, is this response correct? Do I receive 3 places? Or, do I receive 2 places because a developer made a mistake in the SQL query?
7. **Did the client fail to handle the response?**
    - If I receive 3 places back, and everything is fine, maybe I fail to display all the results. Maybe a developer limited the number of entries in a particulat table to be 2, instead of as many as defined by the requirements? That may be a reason for not showing all the places.

It's time to roll up the sleeves and practice with Charles Proxy. The main purpose of Charles is the answer the above captioned 7 questions. We can trace our app activity. In order to to be able to read data, we should go through the installation and configuration steps first. For the Charles to be trusted by the outside world, we need to install the Charles SSL certificate on the computer. Another certificate must be installed of the mobile device. In total, we need 2 SSL certificates - the mobile device and the computer - because they need to communicate and trust each other.

When we open Charles on the desktop, we see some random sequences running, with such as Code, Method, Host, Path, Start, Duration, Size, and Status. At the moment, Charles is trying to log something that is happening on my computer. And, while we can use Charles for web-based app debugging if needed, I want to focus on the mobile devices. In order to get rid of the undesired traffic from my computer, I go to the Proxy menu and uncheck **macOS Proxy**. Afterwards, there should be no interference coming from the computer.

I open an Android emulator, which is already preconfigured, it has its own Charles SSL certifcate installed in addition to the one I have on my computer. I login as a guest to my app. I am not using Postman or anything, I'm just trying to communicate to the back-end of behalf of the app. When I tap the Login button, a few calls get executed. I can highlight the call sequence and view the different data representations or pieces of information I may need - Overview, Contents, Summary, Chart, and Notes. I go ahead and highligt one call:

    Code        Method      Host        Path                            ...
    200         POST        app.name    /api/appName/guest_login        ...

The Overview tab shows the URL that I hit, status complete, response code 200. It also displays timing - when the request and response started and ended, and the duration of communication. It's important to know how long it takes to send a request and receive a response. Consider a scenario when every time I go to *My Places* screen, it takes a long time to load. Of course, I can record a video. And I can also run Charles and get the exact number for how long it takes. I might as well execute multiple runs and calculate the average duration time. When reporting the bug, it's useful to attach the Charles log (.chls file) to evidence how long it took.

Now, moving onto the Contents tab, which is where I spend most of my time. It displays what is being sent to the server:

    {
    “username”: “549w73445675676”
    “captcha”: “appname_android”
    }

Basically, the app is saying that I am trying to login with a randomly generated user, and this user is bypassing captcha. That is a typical flow for my app - in order to login as a guest, I need a randomly generated username stating that I'm bypassing captcha.

And here's what the server responds with (in JSON Text):

    {
    “last login”: 1629856473
    “user”: {
        “renew_token”: “qe876849gnkdfg654768936563hgkl“,
        “token”: “545768hdukhg678sdhkjhyy”,
        “user_key”: “549w73445675676_7572”
            } 
    }

Basically, the server greets the guest user with a welcome, providing me with the user key with the token to be eligible to communicate with the app and so on. The end-point user is not going to see that much.

Let's highlight another call:

    Code        Method      Host        Path                            ...
    200         POST        app.name    /api/appName/userinfo           ...


This looks like that after logging in I go to the *My Account* screen of the app, even though I don't. The *userinfo* bit looks like something that would be requested when a user goes to their account. In fact, even though I do not go to *My Account*, when I login the app actually gets the information about the user right away. It's just programmed that way.

The Contents tab if this *userinfo* endpoint displays the following:

    {
    “user_key”: “549w73445675676_7572”,
    “token”: “545768hdukhg678sdhkjhyy”
    }

The app instantly passes the token that gets generated during login. It allows me to receive the information about the user, the response from the server in JSON Text format:

    {
    “user”: {
            “condition”: “guest”,
            “created”: 1628780774,
            “email”: “guest.549w73445675676@appname.com”,
            “first”: “”,
            “last”: “”,
            “last_accessed”: 1629856473,
            “last_login”: 1629856473,
            “last_modified”: 1629856879,
            “picture”: “https://appname.com/api/appName/pictures/guest”,
            “user_id”: 7572,
            “username”: “guest.549w73445675676”
            }
    }

We are viewing this response for informational purposes. The response in itself is not that meaningful. It merely states that I am a guest and provides my auto generated email which also serves as my unique identifier. Without a unique identifier I cannot login, that's why the system generates those emails. I can also view my user_id, the first and the last time I logged in, and that's basically it.

Now, I go to *My Places* as a guest user. Because I've already logged in as a user a couple of times before, the user details, including *My Places*, are saved on the device. This is the reason why I have places, and it is the call I want to find.

In order to filter out the extra unnecessary information, we use the Filter feature. If we enter "app" or "appname" in the Filter, we only see the calls/entries related to this host name. Upon filtering, let's only focus on the entries related to the app's communication with the back-end. The below captioned entry is the one requesting places:

    Code        Method      Host        Path                                        ...
    200         POST        app.name    /api/appName/places/getuserplaces           ...

I see 2 places instead of 3. While Charles Proxy is running, I go to *My Places* screen. Annd with the help of Charles, I can figure out the user_key and token that were sent to the server (see the Contents tab):

    {
    “user_key”: “549w73445675676_7572”,
    “token”: “545768hdukhg678sdhkjhyy”
    }

**appName/places/getuserplaces** is what I'm requesting. In some apps there would be another "/" and a user_id, but this example app approaches it differently. It sends a sorted username and sorted password, once it hits the server the latter queries the information for my user and returns 3 places - Home (intentionally left blank), Work (intentionally left blank), and Seattle Space Needle (custom location). I can also see the same places on the mobile screen.

    [{
    “address_type”: “home”
    }, {
    “address_type”: “work”
    }, {
    “address”: “Space Needle\n 400 Broad Street, Seattle, Washington, United States, 98109”,
    “address_type”: “custom”,
    “created”: 1620870663,
    “last_modified”: 1620870663,
    “lat”: 47.620422,
    “lon”: -122.349358,
    “name”: “Space Needle”, 
    “place_id”: 2,
    “user_key”: 7572
    }]

JSON is how the data between the API calls is formatted. In the back-end it might send SQL queries in an Oracle database, for example, and the result set gets parsed into JSON and returned to us.

Charles is the "man in the middle" that reveals all the secrets - what was actually sent and what was asked for. It helps us verify that we are not getting what we're expecting to get, when the submitted user_key is something that got cached from a previous user. So, something is incorrect about the user_key, which is why I'm not getting all the content I'm supposed to get. It is a real possibility that can happen with any application.

So, now that I know 1 place is not showing up in the list of places on the UI side, I am ready to file a bug report addressed to the mobile developer. But how can I prove that there is a place that should be visible on the front-end but are not? Perhaps, there is a mistake in the code, and some or all of the custom places are not showing. Certainly, I can support the bug report with a screenshot and a result set fo the SQL query, but it's insufficient. I generate a Charles log and attach it to the report. To do that, we can go to File and select an option Save Session or Save Session As, and save it as log.chls file. Now we can attach this file to JIRA, which support the .chls file format. When the developer receives the report, they download the log.chls file, double-click it, and the file opens in the Charles app itself. The developer is able to view everything we experience when we test the app. The developer look at the call **appName/places/getuserplaces** accessing *My Places*, and now they are convinced because the Charles log shows Space Needle coming from the back-end but not showing in the app's UI. 

Collecting Charles logs is a piece of cake for testers and developers. It's a simple and helpful feature.


Once again, one may ask "Why not use Postman for activity logging and tracking?". A few years ago, Postman introduced the [Postman Interceptor](https://learning.postman.com/docs/sending-requests/capturing-request-data/interceptor/#:~:text=Postman%20Interceptor%20is%20a%20Chrome,bound%20session%20of%20traffic%20capture.), which attempts to accomplish the same thing as Charles Proxy. We can turn it on and record the traffic communication between the web or mobile app and the back-end. If you choose to explore the [Interceptor](https://blog.postman.com/postman-interceptor-the-next-generation-view-source-for-the-api-economy/), make sure to use with the desktop version of Postman. It doesn't work with the Postman web app.

I personally do not use the Interceptor because **Logging** only comprises 20% of the time I spend on Charles. The other 80% of tasks and features, such as Re-Write, Map Local, Throttling, and Breakpoints are covered in the following sections.

## Throttling

We can change Network Conditions using Charles. When testing software on mobile devices, we run into an issue with physical Android devices. While we can mimick various Network Conditions on a virtual Android emulator, doing the same on an actual Android device requires a tool like Charles Proxy.

In the Proxy menu there are 2 related options - "Throttle Settings" and "Start Throttling". First, let's review Throttle Settings, which look very similar to the iOS device configuration. Once we "Enable Throttling", we can choose to change network conditions for All hosts that we are hitting (https://*:*), which means every application that is visible to Charles Proxy.

We can be more specific and tag a particular app, for which we'd have to provide the host name. Then we can change the network conditions by using "Throttle Preset" and selecting a network type - 3G/4G/Fiber/..., and defining an own profile if desired. The **Throttling** feature in Charles is one of the options that help with testing network conditions on Android apps.

Charles also has 3 other features that I love - Map Local, Re-Write and Breakpoints. I love using Charles for mobile app testing, but these features can be used web-based apps as well.

## Map Local

Let's consider a scenario where I have a requirement that allows up to 30 places under *My Places*. It means I can add 30 places max, and the app should display those places accordingly. To test boundary conditions, I use the test data of 30 and 31 days.

Some questions that come to mind in this test scenario are: Where does the 31st place go? Does it replace any of the existing 30 places? Does the '+' Add button get disabled, once 30 places are added - which is not such a good user experience? Or, if I tap the '+' Add button when creating the 31st place, do I get a popop message or an alert stating something like "30 is max. You've reached the max. Please delete something"?

With the test data and my test case in place, I can now go ahead and manually create the 30 places in the app. I tap the '+' Add button, search for a place/address, name the selected location, and tap Save. I expect some call to be happening in the back-end to update the database, such as:

    Code        Method      Host        Path                        ...
    200         POST        app.name    /api/appName/places/add     ...
    

Or, instead of manually creating place one by one, I can test this scenario with the help of Postman. I could run the same request and send information with a bunch of places to the back-end, so they all get added. It's a reasonable approach, which I could use to add the 31st place. However, I should not forget to delete those places upon test execution. It may get complicated. Once a record is inside the database, it's inside, unless I delete it. Sometimes I have to delete everything, but keep Home and Work, so I have to figure out how to only delete the intended places from the database. It can be inconvenient and time-consuming.

We could generate the data with Postman, or could actually add the data to the database directly by using the UPDATE command. But what if we don't want to save anything in the database? My main intent is to verify that I can fit 30 places on the *My Places* screen, that it is scrollable and all, and that the 31st place does not get added.

We can fool an app, because Charles as the 'man in the middle' has the beautiful ability to intercept requests and responses, decrypt them, and display them in the Charles UI. Charles receives a response from the server and then sends it to me and the app. We can see the call details in the desktop Charles app as well as the mobile device screen.

Because Charles sits as the 'man in the middle', we can change the server response before forwarding it. We can take control and ask Charles to not send something real that's coming from the server but replace it with fake data. Ib this case, we are not inserting anything into the database, not creating any places, we're just fooling an app.

I log out as a guest and log back in as a registered user. I go to *My Places* in my app. Then in Charles I click on the "getuserplaces' call sequence:

    Code        Method      Host        Path                                ...
    200         POST        app.name    /api/appName/places/getuserplaces   ...

The Contents tab display the following user details:

    {
    “user_key”: “lana_sdet@gmail.com_2783”,
    “token”: “36uw5768hdukhg678sdhhg6”
    }

Once this request hits the server, the server queries the information for the user and returns the user's Home, Work, and custom Space Needle location. The same places display on the mobile screen as well.

    [{
    “address_type”: “home”
    }, {
    “address_type”: “work”
    }, {
    “address”: “Space Needle\n 400 Broad Street, Seattle, Washington, United States, 98109”,
    “address_type”: “custom”,
    “created”: 1620870663,
    “last_modified”: 1620870663,
    “lat”: 47.620422,
    “lon”: -122.349358,
    “name”: “Space Needle”, 
    “place_id”: 2,
    “user_key”: 7572
    }]

The **Map Local** feature of Charles Proxy can help me make Charles send to my app not the actual real response that comes from the back-end but something else entirely. We can 'fool' the application with the fake 'getuserplaces' data, for example.

Go to the Tools menu, check the Map Local Settings option, and Enable Map Local. In the Map Local, we can manipulate the app, when it hits a particular endpoint, to diplay some other data in lieu of the real actual data coming from the back-end server. I need to fool the app with the 'getuserplaces' part.

I copy the call's URL, go to Tools > Map Local Settings, and then add this endpoint. The menu shows Edit Mapping and has sections for Map From and Map To. In Map From, we have to provide our protocol type (http or https), our hostname (appname.com), our port number (default port is 443 - can leave it empty), and provide the path (/api/appname/places/getuserplaces). Don't populate the fields individuall, just paste the entire endpoint URL into any field, click inside any other field, and all the respective fields will auto-complete.

Now, in the Map To section, we have to provide a "Local Path". If I click Choose, I should be able to upload a file. What kind of file? I want to upload a JSON file, where I have a different data set. Instead of 3 places - Home, Work, and Space Needle - I need to test more. So, I copy whatever server response I have for now and paste it in a text or code editor. Then I copy/paste paste the Space Needle address multiple times, until I end up with an array of 30 JSON objects. I customize the address values with a numeric index to distinguish among them. During testing, this helps to tell how many hit the screen and identify them. 

So, let's create multiple places and save the file as multi_reservations.json (need JSON format files for Charles):

    [{
    “address_type”: “home”
    }, {
    “address_type”: “work”
    }, {
    “address”: “Space Needle1\n 400 Broad Street, Seattle, Washington, United States, 98109”,
    “address_type”: “custom”,
    “created”: 1620870663,
    “last_modified”: 1620870663,
    “lat”: 47.620422,
    “lon”: -122.349358,
    “name”: “Space Needle”, 
    “place_id”: 2,
    “user_key”: 7572
    },
     {
    “address”: “Space Needle2\n 400 Broad Street, Seattle, Washington, United States, 98109”,
    “address_type”: “custom”,
    “created”: 1620870663,
    “last_modified”: 1620870663,
    “lat”: 47.620422,
    “lon”: -122.349358,
    “name”: “Space Needle”, 
    “place_id”: 2,
    “user_key”: 7572
    },
    
    ...
    
     {
    “address”: “Space Needle27\n 400 Broad Street, Seattle, Washington, United States, 98109”,
    “address_type”: “custom”,
    “created”: 1620870663,
    “last_modified”: 1620870663,
    “lat”: 47.620422,
    “lon”: -122.349358,
    “name”: “Space Needle”, 
    “place_id”: 2,
    “user_key”: 7572
    },
    …
     {
    “address”: “Space Needle28\n 400 Broad Street, Seattle, Washington, United States, 98109”,
    “address_type”: “custom”,
    “created”: 1620870663,
    “last_modified”: 1620870663,
    “lat”: 47.620422,
    “lon”: -122.349358,
    “name”: “Space Needle”, 
    “place_id”: 2,
    “user_key”: 7572
    }]

So, we want to fool the app. We want Charles to send to the app the data which we've just defined in the multi_reservations.json file, instead of what actually got retrieved from the database. Let's right-click on the call, copy the URL, go to Tools > Map Local, and click Add. The Edit Mapping menu opens. In the Map From section, paste the URL and click outside. In the Map To section, we want to choose the multi_reservations.json file from the local machine. Provide the local path, click OK, and then OK again. Make sure that in the Tools menu, [Enable] Map Local option is checked/activated.

Now, in my transportation app, I navigate outside of the *My Places* screen and navigate back to it again. I.e., go to the *My Account* call /api/appname/userinfo in Charles, the go to the *My Places* call /api/appname/places/getuserplaces again. And ta-da! The mobile screen diplays all the places added to the JSON file.

When we need to run such a test case, we can just attach the JSON file to the test case and use it every time we need it. We don't have to create, insert, or delete anything afterwards, we can just 'fool' the app. The whole idea is to be able to test the UI side - verify we can see 30 places, we do not see the 31st place, and that the view is scrollable. With the help of Charles is super simple to pass data to the app on behalf of the back-end.

Map Locals stay recorded in Charles. You can de-activate them individually or in bulk. You don't have to recreate them every time, only to select or deselect specific "local files to serve remote locations" for testing.

At the end of the test, I have two JSON files with 30 and 31 places respectively. When done testing, I uncheck the Map Local boxes that I don't need any longer. So, I uncheck my local JSON file in Map Local, so that I can see the regular *My Places* screen with the 3 places I actually have - Home, Work, and Space Needle.

Web app testing would work the same. The below examples are for testing web-based apps:
- Modify Requests, Responses, Status Codes, etc.
    - https://medium.com/@IlyaEremin/modify-api-response-for-android-app-with-charles-181a822cfc24
    - http://www.testeffective.com/better-mobile-app-testing-with-charles-proxy/
    - https://www.thinktecture.com/en/tools/debugging-proxies-mocking-manipulating-api-charles-in-action/ 


## Re-Write

The **Rewrite** feature is somewhat similar to Map Local, but it's more useful when testing something small, i.e. a small piece of data. Rewrite achieves the same results, but may be less convenient than Map Local. With Map Local, we can just work with a file which is easier to read. With Rewrite, we copy/paste everything in one field.

Just as Map Local, we can also locate the Rewrite feature under the Tools menu. Have it checked to activate the feature. We can open the Rewrite Settings, which modify requests and responses as that pass through Charles, and turn on/off a particular feature in the application under test. For example, in my transportation app, there is a *Stored Account* feature, the Signin screen has a "Stored Account' button at the bottom. When I tap on it, I get the message "not yet available". Maybe, it is or will not be available at all. Maybe it will be available, but has not been thoroughly tested yet and should not ne shown to the user. In this case, the app can hide a particular feature behind a feature flag.

Rewrite in Charles is the only tool for me to be able to turn on/off a particular feature. Set it to True to display a feature to the user, set it to False to not show it.

Another popular use for Rewrite, is testing the scenario when the server returns a 500 status. So, let's say, I have to test how the app handles Server Errors, because sometimes servers fail and testers observe a 500 error. We want to make sure that the app does not crash if the server fails. We must specify the location/enspoint we're going to rewrite. This time, I'm going to rewrite the User Info /api/appname/userinfo. I copy the call's URL and give it a the name UserNameServerError.

Let's click Add and observe the Edit Location menu appear with fields, such as Protocol, Host, Port, Path, and Query. We can populate them manually, Or, we can paste the URL and click outside of the field so that it auto-populates, and then click OK. Now, the path shows in the Location section of Rewrite Settings. This indicates to Charles that every time the app executes this request, we need to make some modifications. What modifications? We can define them by tapping Add on the bottom of the Rewrite Settings menu. Tbe Rewrite Rule menu opens, and we can modify a bunch of things. The Type dropdown menu has options Add/Modify/Remove Header, Host Path, URL, Add/Modify/Remove Query Param, Response Status, and Body. Most often, I modify the Request and Response Body.

We can choose to Rewrite the Body, but then we have to provide everything we want to rewrite in the Value field. This field is not limited, but it's inconvenient to edit. I cannot see the 'whole picture' of the test data, as I can by reading a file with Map Local. That's why, again, I prefer to use Rewrite to turn a feature on or off. But let's get back to our Server Error scenario.

We are testing a scenario when the app request the information about a user, and the server returns the 500 status. So, we do not rewrite the response Body, we rewrite the response Status. We cannot change the Status with Map Local. To replace the real information coming from the back-end server, I input 500 in the Values field and tap Save. Back in the Rewrite Setting, we now see Type:Status, Action:500.

Let's check what happens with the transportation app. I restart it and login with the existing user credentials. I'm inside my mobile app, but the the code that is showing for the 'userinfo' call is 500 now:


    Code        Method      Host        Path                        ...
    💣500       POST        app.name    /api/appName/userinfo       ...

The code has changes from 200 to 500. We are experiencing a Server Error. If I go to *My Account* I see no information about the user. The server does return something, but because of the 500 Server Error the app is not consuming it. The app thinks that the server response merely arrived with an error. Thanks to that, the app doesn't show any of the "userinfo", but at least it's not crashing.

Checking how the application handles server errors is a very valuable scenario. It's crucial to verify the app does not crash when it receives the 500 Server Error. When the server is down, we expect a generic user message saying something like "something went wrong, please try again". When a user sees an empty *My Account* view, they do not understand what's happening. Hence, on the UI side it's useful to generate a toast message notifying of a problem. The 500 errors can be very generic and hard to isolate but they do happen, and the app does not know the specific nature of the problem.  That's why a generic UI message would suffice to verify the app handles such errors gracefully.

To verify the outcome of Server Errors without a tool like Charles Proxy would mean asking the back-end developers to shut down the QA server. This would be an unfeasible and unprofessional approach to this testing scenario.

When done testing, remember to uncheck the [Enable] Rewrite, individually or in bulk.

## Breakpoints

Now, let's explore the **Breakpoints** feature with a different mobile application. It's an advertising/marketing platform for various commercial brands like Uniqlo, Target, etc., to post videos, surveys, and trivia questions for the users to complete/view and earn points. Later the user can redeem these points, for example, as gift cards for movies.

Different apps have different goals, but a lot of apps are trying to make users return and stay as long as possible. The AUT in this example tries to improve user retention, it has a new feature called *My Streak*. It is one of the ways to boost up the user points by motivating the users to visit the app more often. The app offers extra points if a user logs in a few days in a row. If I check in for 4 days, I get a 5% bonus, 15 days - a 10% bonus, and 30 days - a 15% bonus.


The *Streak* screen states "In 3 days you'll get a +5.0% bonus on all point earnings!" This is a motivation for a user to come back. In my case, it's 3 days (not 4) because I already have a 1-day check-in.

<img width="500" src="https://github.com/lana-20/charles-proxy/assets/70295997/4b8a6c23-fc8a-4814-9a0e-ed92bfe5f58a">


In Charles Proxy, let's type "appname" in the Filter field to focus on the AUT. Here are the relevant call sequences:

    Code    Method  Host                    Path					…
    200     GET     api.appname.com         /app/v1/user/affiliate/list		…
    200     GET     api.appname.com         /app/v1/streak				…
    200     GET     api.appname.com         /app/v1/balance     			…
    200     GET     api.appname.com         /app/v1/user        			…

*Streak* is a new feature, and I am assigned to test the user story. One of the acceptance criteria in the user story is that when a user hits 4+ days (+5.0% bonus), the progress bar should turn orange up to the next boundary on the range slider. When a user hits 15+ days (+10.0% bonus), the progress bar should turn orange up to the next boundary again. If a user visits the app for 30+ days in a row and reaches the +15.0% bonus, the bar should turn orange rightwards up to the next boundary once again. So, the test data is: 0, 1, **4**, 5, **15**, 16, **30**, 31 (boundary values plus one single in-between value for every range).

Let's look at the Contents tab, Headers subtab for the "/app/v1/streak" call. At the very end there is a summary:

    …
    }],
    “user”: {
    “id”: “eellaa4d-4a25-9b0d-af38ce246dc0”,
    “userId”: “21093984-b09b-4ae5-95ad72eaa0d”,
    “streakDate”: “2023-06-12”,
    “streakCount”: 1,
    “bonus”: 0.00,
    “earning”: null,
    “credited”: null,
    “checkInDate”: 1686592074094,
    “earningDate”: null,
    “checkedIn”: true,
    “currentStreakTotal”: null,
    “lifetimeStreakTotal” null:
    }
    }

If we test the test case manually, we'll be here for a month. We don't have 31 days to spend on waiting, when our sprints are 1-2 weeks long. Instead, we should use the Charles Breakpoints feature and complete the task in 30 minutes.

**Breakpoints** pause the execution, which we see in automation tests. Developers and SDETs use breakpoints when they debug a program to see what's happening at a particular execution step. Charles can pause execution of our requests and responses, and we can modify those in real time.

#TODO - finish Charles practice notes and illustrations






___

- [What is Proxy? How does it work?](https://github.com/lana-20/proxy-server#readme)
- [What is (HTTP) Proxy?](https://github.com/lana-20/ssl-tls-http-https/blob/main/README.md)


Charles Proxy Tutorials:

- [Charles Proxy Tutorial for iOS](https://www.raywenderlich.com/1827524-charles-proxy-tutorial-for-ios)
- [Karl von Randow, creator of Charles about Charles for iOS app](https://www.youtube.com/watch?v=RWotEyTeJhc )
- [Charles Proxy Tutorial for Android](https://medium.com/@daptronic/the-android-emulator-and-charles-proxy-a-love-story-595c23484e02)
- ['Hack' an Android app to edit manifest file](https://medium.com/@ferrygunawan/debug-ssl-traffic-with-charles-proxy-872aedb228d)

Modify Requests, Responses, Status Codes, etc.:

- [Modify API response for Android app using Charles proxy](https://medium.com/@IlyaEremin/modify-api-response-for-android-app-with-charles-181a822cfc24)
- [Better Mobile Application Testing with Charles Proxy](http://www.testeffective.com/better-mobile-app-testing-with-charles-proxy/)
- [Charles Proxy In Action: Mocking And Manipulating API Behavior With A Local Proxy Server](https://www.thinktecture.com/en/tools/debugging-proxies-mocking-manipulating-api-charles-in-action/)

Status Codes and Protocols:

- [HTTP Status Codes](https://www.restapitutorial.com/httpstatuscodes.html)
- [WSS (WebSocket Secure) Protocol (used by messengers apps like Slack)](https://devcenter.heroku.com/articles/websocket-security)

Notes:

- [Notes on Charles Proxy](https://github.com/lana-20/charles-notes)
- [Configuring Charles Proxy](https://github.com/lana-20/charles-setup)




