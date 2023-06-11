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

...

## Re-Write

...

## Breakpoints

...


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




