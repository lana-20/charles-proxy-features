# [Charles Proxy](https://www.charlesproxy.com/overview/about-charles/) Debugging Tool for Mobile App Testing

<img src="https://user-images.githubusercontent.com/70295997/226082568-0a72d438-99c3-4962-8e16-724df840d581.png" width=600>

Almost every software product, whether it’s web or mobile, has some kind of tracking implemented. Tracking is used for gathering insights on how the product is used. User flows can be tracked to improve the way users are working with the product. However, while user tracking can be useful it can also be misused.

If a mobile app is using tracking, the mobile team should ask themselves if it's really necessary to track and if it's helpful for the business? There are laws in many countries that protect user data and how it’s used. The EU General Data Protection Regulation (GDPR) is the law focusing to consolidate the data privacy laws across the EU. Companies have to inform their users about the implemented tracking and why and how the data is used. If a company does not follow the law, it can be sued and fines can be assessed for violation of the law. Whenever tracking is implemented, mobile teams should know the regulations for user tracking and data, so they do not open their company to liability inadvertently.

If tracking is in place, make sure it works before every release. Use tools like Charles Proxy or Fiddler to verify. Charles Proxy is what I've been using as a tester. When conducting Mobile App Testing, I have Charles running on my machine 3/4 of the time.

For *Android* versions of the app *only*, the developers provide testers with a special debuggable build that we can use with Charles Proxy. They add a Network Security Configuration file into the Android project in order for us to be able to read the traffic in Charles. The production versions do not have this file. We only have it in the test versions, the debug builds, and only if the developers have added the configuration file. 99% of the time the test version of the app is already configured for use with the Android version. iOS does not require that, only Android does. Without the special debug build, we aren't able to view the traffic.

Besides tracking, the **Logging** feature of a proxy server is instrumental in debugging issues during the root cause analysis. It helps us pinpoint a failure to either the front- or the back-end. Mobile Testing often involves utilization of proxies to read the traffic between the app's front-end & back-end (and any nook & cranny, ebbs and flows that a request or response may traverse through). For extensive logging and root cause analysis, I use Charles Proxy. 

The request or response can encounter issues in many different places, while travelling over HTTP from the client to the server and back. What are those places?

Karl von Randow, a New Zealander, an experienced developer who started more than 20 years ago when Chrome or Safari did not exist yet. There were no good debugging tools. Nowadays, we can check for any issues with our Requests in the browser DevTools (for a web-based app). We can inspect the page, analyze multiple tabs such as Elements, Console, Sources, Network, Performance, Memory. We can go to our Network Tab and figure out which end-point we was trying to hit, our status code, how much time it took, etc. We can check it in the DevTools, but Karl couldn’t. At that time, there was no such thing as Chrome DevTools, there was no Chrome. He created the ‘man in the middle’ who’d help him to debug his web-based app. For example, to figure out why he did not see things that he would expect to see, and what was failing. Not having a debugging tool may cause a lot of frustration. So, he created Charles Proxy.

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

Postman is not the ‘man in the middle’. It has no idea what’s going on on the front-end. It's just a tool that lets us, as testers, play the role of the app and request data from the server on behalf of the app. 
But if we want to know more and get answers to all of the 7 questions, we're looking into using a secure proxy server to intercept (listen in on) the traffic.

Privacy, speed, bandwidth savings, activity logging are the benefits of using a Proxy Server. Charles Proxy takes advantage of one of the abilities of proxy servers. This ability is activity logging. Charles sticks in the middle, records communication activities between client and server -- and we can view it in real time, it helps us identify and isolate the issue.

When can such activity logging be useful? Let's say, you open an app and see less content that you expect. How would you approach that?

For example, I am working on an app that reserves public transportation for visually impaired users. In this app, I can save various places to *My Places*. For testing purposed. I have 5 places saved already. But when I navigate to *My Places* screen, I only see 3. There's an obvious issue here. How to approach it? I could check the database and get a result set of 5. That would mean a UI issue, and I'd file a bug report with my mobile developers not the back-end ones.

However, things can go wrong in many places, there is a long way between the UI and the database. I could query the database and get 5 places in return. But that would not immediately indicate a UI bug. In order to be sure, I need to closely trace how my app's UI side communicates with the backend. Apparently, when I login I do not query the database directly. There are all sorts of API methods - GET, PUT, POST, PATCH, DELETE. When I look for *My Places*, the app makes an API request to GET places for the user. I know which users are logged in when I'm in my app. For a particular logged in user, the app asks the back-end to provide their places. Then the back-end goes and retrieves those places from the database. Behind the scenes, it means that once the app executes this request, asking for places for a specific user, the back-end selects places from the Places table:

SELECT place from Places
WHERE user_id = ' ... '

There is a chance that a developer made a mistake when composing a query. They may forget something or not GROUP by something. We might have to query multiple tables. The developers are human, they can make mistakes.





___

[What is Proxy? How does it work?](https://github.com/lana-20/proxy-server#readme)

[What is (HTTP) Proxy?](https://github.com/lana-20/ssl-tls-http-https/blob/main/README.md)

___

Charles can be installed in our machines as a desktop app. We can also install it on iOS device without connecting Xcode, but the mobile app has some limitations. The iOS version of Charles covers some extra scenaros, however usually I use the desktop app which is much more feature- and functionality-rich.

Charles configuration may be a bit complex, but its benefits are more than worth it. There's a 30-day free trial, after which a $50 life-time license is available for purchase. Without a license after a free trial, Charles still works but it shuts down every 30 minutes, which is usually enough to test. You can just restart it and keep testing.

[Configuring Charles Proxy](https://github.com/lana-20/charles-setup):

- [Mac]()
- [Windows]()
- [Mac and iOS device](https://youtu.be/vtSLoCC299U)
- [iOS device]()
- [Android device]()
- [Android emulator](https://youtu.be/WJYf9nkSIKA)
  - Sometimes when installing the certificate on an emulator, the page [chls.pro/ssl](chls.pro/ssl) is not loading. This tutorial (TBD), shows an alternative way of installing the certificate on the emulator. 
Also you may try to head over to [charlesproxy.com/getssl](charlesproxy.com/getssl) if the short url [chls.pro/ssl](chls.pro/ssl) doesn’t work.


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




