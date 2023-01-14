# Charles Proxy Debugging Tool for Mobile App Testing

Almost every software product, whether it’s web or mobile, has some kind of tracking implemented. Tracking is used for gathering insights on how the product is used. User flows can be tracked to improve the way users are working with the product. However, while user tracking can be useful it can also be misused.

If a mobile app is using tracking, the mobile team should ask themselves if it's really necessary to track and if it's helpful for the business? There are laws in many countries that protect user data and how it’s used. The EU General Data Protection Regulation (GDPR) is the law focusing to consolidate the data privacy laws across the EU. Companies have to inform their users about the implemented tracking and why and how the data is used. If a company does not follow the law, it can be sued and fines can be assessed for violation of the law. Whenever tracking is implemented, mobile teams should know the regulations for user tracking and data, so they do not open their company to liability inadvertently.

If tracking is in place, make sure it works before every release. Use tools like Charles Proxy or Fiddler to verify.

Besides tracking, Charles Proxy is instrumental when I encounter issues in many different places. What are those places?

1. Did I send a request?
2. Was the request correct?
3. Did the server fail?
4. Did the network fail?
5. Did I receive a response?
6. Was the response correct?
7. Did the client fail to handle the response?

With Postman, I can only answer questions 5 and 6. Did I receive the response, and was it correct? 

Postman is a great tool, helpful in situations such as:
- I have the back-end ready, but the UI is not done yet. I can test features early, check if the API works well, before it gets connected to the UI.
- Create a Collection to help me generate data. Let's say, to run Regression Testing, I need an account that has multiple reservations and multiple places. Creating this data manually is tedious. I can create Postman Collections that generates the data for testing. A Collection can create a few places and make a few reservations for me.

But I dont' exclusively work with the back-end, when testing a mobile a web-based app. I do use Postman, but not as often as one might think. Once in a while, I might run some Collections or run the requests one by one.

Postman is not the ‘man in the middle’. It has no idea what’s going on on the front-end. It's just a tool that lets me, as a tester, play the role of the app and request data from the server on behalf of the app. 
But if I want to know more and get answers to the 7 Questions, I’m looking into using a secure proxy server to intercept (listen in on) the traffic.

Karl von Randow, a New Zealander, an experienced dev who started more than 20 years ago when Chrome or Safari did not exist yet. There were no good debugging tools. Nowadays, I can check for any issues with my Requests in the browser DevTools (for a web-based app). I can inspect the page, analyze multiple tabs such as Elements, Console, Sources, Network, Performance, Memory. I can go to my Network Tab and figure out which end-point I was trying to hit, my status code, how much time it took, etc. I can check it on the DevTools, but Karl couldn’t. At that time, there was no such thing as Chrome DevTools, there was no Chrome. He created the ‘man in the middle’ who’d help him to debug his web-based app. To figure out why he did not see things that he would expect to see, and what is failing, eg. Not having a debugging tool may cause a lot of frustration. So, he created Charles Proxy.

[What is Proxy? How does it work?](https://github.com/lana-20/proxy-server#readme)

[What is (HTTP) Proxy?](https://github.com/lana-20/ssl-tls-http-https/blob/main/README.md)

Privacy, speed, bandwidth savings, activity logging are the benefits of using a Proxy Server. Charles Proxy takes advantage of one of the abilities of proxy servers. This ability is activity logging. Charles sticks in the middle, records communication activities between client and server, and I can see it in real time. It helps me identify the issue.

Charles configuration may be a bit complex, but its benefits are more than worth it. There's a 30-day free trial, after which a $50 life-time license is available for purchase. Without a license after a free trial, Charles still works but it shuts down every 30 minutes, which is usually enough to test. I can just restart it and keep testing.





