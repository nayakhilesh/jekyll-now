---
layout: post
title: Memorable Bugs
---

Some of the more memorable bugs I've encountered over the course of my career thus far:

## Process.java
We had a java application that would periodically use the java Process API to ssh into a remote server, list files and retrieve some of them.
This process would ocasionally hang - the buffered reader trying to read the list of files would hang indefinitely in the read call. We found only one obscure online forum post about a similar issue from over 5 years prior that had no resolution. Ultimately due to time pressure we just moved that portion of the functionality to a bash script and called it instead. Worked like a charm.

## Map.get(Object key)
In java Map.get takes an object and not a generic argument. This means that even if you have refined the type of the Map using generics (e.g.: Map<String, String>) you will not get a compiler error for using a key of a different type as an argument to get. I had a colleague spend 2 months (while also working on other things of course) trying to find a bug in our distributed caching infrastructure that was caused due to the use of a String argument to get instead of the appropriate Enum. Unit testing and static code analysis would probably have caught such a bug much earlier. [A better API for the get method (with a generic argument) would be ideal - but would break backwards compatibility with pre-generics java]

## Caching
I wrote a bunch of filtering, indexing, on-demand refresh and async update functionality for a multi-tier caching solution developed in-house and used by our application. Found a few bugs only after writing 20+ unit tests for it. This is the best time to find bugs!

## Backup != Restore
In case the cache process ever went down there was a command line option that would instruct it to bootstrap with data from the DB on startup. The data managed by the application was refreshed from a separate canonical source thrice a day but if the cache went down intra-day between these refresh times then we would need to boostrap it with data from the DB. I discovered that the bootstrapping functionality for some sections of data never worked. A backup isn't really one unless you've tried to restore from it.

## Heisenbug
We had an integration test that would fail between 8 and 9pm everyday. A colleague and I were the only ones to notice it since we would adhere to a later work schedule. I don't recall the specifics but it was due to daylight savings time (DST).

## Time travel
One of the integration tests was failing for me despite doing a fresh checkout of the code and setting up my test database again from scratch. On debugging I found that it was failing in a stored procedure due to drift of the clock on my Linux test DB host.

## Back pressure
The integration test suite started failing at the same test every time they were run. All tests after a certain test failed. On debugging I found that the web server thread was hanging in a log statement. We had recently made an addition to a config file to have the JVM that runs the tests log to stdout - but this change was also being picked up by the application and so it was also trying to log to stdout. The application JVM was being started in a Gradle build file using groovy's ArrayList.execute method which unknown to us was using java's Process API under the covers. Since we weren't consuming the launched process's output it was blocking after writing a specific amount of data to the output buffer. A simple one line addition "process.consumeProcessOutput()" and everything was right again. [By now it might have become somewhat clear that the java Process API isn't too fun to work with :(]

## 2nd law of thermodynamics
We were seeing really slow startup times for our 8 JVMs on our Jenkins build slaves. Sometimes they wouldn't be up even after 20 minutes. On debugging we found that the main thread would never return after it went into code trying to fill the DB connection pool. The devops engineer and I were both stumped - we were able to use strace to figure out that the main thread was waiting on something. The credit for figuring out this issue goes to one of my colleagues who found a post online about a similar issue and discovered that the Oracle driver was waiting trying to read /dev/random - since the Jenkins slaves were low on entropy when all the VMs were being started simultaneously and all trying to fill their connection pools they were all waiting. We had to add a daemon process to the server that would generate additional entropy.

## Prometheus
Some data wasn't being populated on our customer service UI. The cause was traced to calls made to our backend application that were timing out. Taking a look at the logs we were able to tell that these requests were taking over 30+ mins to return in some cases. A quick scan of the code alongside the logs showed that we were making multiple calls to the DB to retrieve some information for each of the results of a query we had made previously (a classic N+1 select). Git blame showed us that the code for these queries had been introduced recently since a colleague wanted to normalize the response across 2 of our endpoints and had thus enhanced the endpoint being called to include a few more fields (which required the additional query). We had forgotten that the list of items for which we were running the additional queries could be so large (in the tens of 1000s) - the normal use case was a small list - these outliers were due to a hack that was put in place to enable a new business process a few years prior and that had been deprecated but the old data still existed.

## Spoofing
A user had tried to link a subscription with a spoofed itunes receipt - she had changed it to a valid purchase receipt when the request was in flight from the device - this valid receipt was successfully validated by AAPL and thus gave the user access to the product when they didn't actually buy it. My colleague fixed it by changing the code to decode the base64 encoded receipt and check the product id in the receipt was correct.

## Dead on arrival
When crediting a user in some cases a portion of the credit needs to go back to the tax and discount accounts for accounting purposes. I was trying to make sense of the code doing this in order to better understand how it was working as part of a project to do an implementation of a new financial reporting and acounting system. The variable representing the fraction that the credit was of the discounted invoice amount [helpfully called 'percents' :(] was being set incorrectly in an if clause which was not entered very often (when the credit to be given was larger than the max credit still allowed to be given on the invoice). Looking at past versions of the code it became quite apparent that the bug had been in place for over 6 years and the code had always been buggy. I had needed to refactor and rename the (quite indecipherable) code to actually understand what it was doing before looking at correct data and comparing with the incorrect data to identify the bug. Have I mentioned it's a good idea to write tests?

## Back to the future
The integration test suite started failing in our CD pipeline. On investigation I found another DST bug - Joda Time advanced the clock by an hour on adding the number of minutes in 14 days (to now) since the 14 days ended after March 12th (when DST starts). The sql query doing the relevant filtering on the application side did not add the extra hour since it was just adding the 14 days in Oracle sql. This led to the record that the test expected the application code to pick up being dropped and thus not being processed and failing the test asserts.


