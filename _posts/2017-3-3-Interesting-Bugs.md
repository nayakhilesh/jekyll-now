---
layout: post
title: Interesting Bugs
---

Some of the more interesting bugs I've encountered over the course of my career:

## Process.java
We had a java application that would periodically use the java Process API to ssh into a remote server, list files and retrieve some of them.
This process would ocasionally hang - the buffered reader trying to read the file would hang in the read call. We found only one obscure online forum post about a similar issue from over 5 years prior that had no resolution. Ultimately due to time pressure we just moved that portion of the functionality to a bash script and called it instead. Worked like a charm.

## Map.get(Object key)
In java Map.get takes an object and not a generic argument. This means that even if you have refined the type of the Map using generics (e.g.: Map<String, String>) you will not get a compile error for using a key of a different type as an argument to get. I had a colleague spend 2 months (while also working on other things of course) trying to find the bug in a distributed caching application that was caused due to the use of a String argument to get instead of the appropriate Enum. Unit testing, languages with more expressive type systems and static code linters would all probably have caught such a bug much earlier.

## Caching
I wrote a bunch of filtering and indexing functionality for a multi-tier caching solution developed in house and used by our application. Found a few bugs only after writing 20+ unit tests for it.

## Heisenbug
We had an integration test that would fail between 8 and 9pm everyday. A colleague and I were the only ones to notice it since we would adhere to a later work schedule. I don't recall the specifics but it was due to daylight savings time (DST).

## Time travel
One of the integration tests was failing for me despite doing a fresh checkout of the code and setting up my test database again. On debugging I found that it was failing in a stored procedure due to the drift of the clock on my Linux test DB host.

## Back pressure
The integration test suite started failing at the same test every time they were run. All tests after a certain test failed. On debugging I found that the JVM was hanging in a log statement. We had recently made a change to a config file to have the JVM that runs the tests log to stdout - but this change was also being picked up by the application and so it was also trying to log to stdout. The application JVM was being started in a Gradle build file using groovy's ArrayList.execute method which unknown to us was using java's Process API under the covers. Since we weren't consuming the launched process's output it was hanging on writing to stdout. A simple one line addition "process.consumeProcessOutput()" and everything was right again.

## Entropy
We were seeing really slow startup times for our 8 JVMs on our Jenkins build slaves. Sometimes they wouldn't be up even after 20 minutes. On debugging we found that flow of control would never return after it went into trying to fill the DB connection pool. The devops engineer and I were both stumped - we were able to use strace to figure out that the main thread was waiting on something. The credit for figuring out this issue goes to one of my colleagues who found a post online about a similar issue and discovered that the Oracle driver was waiting trying to read /dev/random (presumably as part of some encryption or handshake process) - but since the Jenkins slaves were low on entropy when all the VMs were being started simultaneously and all trying to fill their connection pools they were all waiting. We had to add a daemon process to the server to generate additional entropy.

## Repitition
Some of the data wasn't being populated on our customer service UI. The cause was traced to calls made to our backend application that were timing out. Taking a look at the logs we were able to tell that these requests were taking over 30+ mins to return in some cases. A quick scan of the code alongside the logs showed that we were making multiple calls to the DB to retrieve some information for each of the results of a query we had made previously (a classic N+1 select). Git blame showed us that the code for these queries had been introduced recently since a colleague wanted to normalize the response across 2 of our endpoints and had thus enhanced the endpoint being called to include a few more fields (which required the additional query). We had forgotten that the list of items for which we were running the additional queries could be so large (in the tens of 1000s) - the normal use case was a small list - these outliers were due to a hack that was put in place to enable a new business process a few year prior and that had been deprecated but the old data still existed.

## Spoofing
A user had tried to link a subscription with a spoofed itunes receipt - she had changed it to a valid purchase receipt when the request was in flight from the device - this valid receipt was successfully validated by AAPL and thus gave the user access to the product when they didn't actually buy it. We fixed it by changing the code to decode the base64 encoded receipt and check the product id in the receipt was correct.

## Dead on Arrival
When crediting a user in some cases a portion of the credit needs to go back to the tax and discount accounts for accounting purposes. I was trying to make sense of the code doing this in order to better understand how it was working as part of a project to do an implementation of a new financial reporting and acounting system. The variable representing the fraction that the credit was of the discounted invoice amount (helpfully called 'percents') was being set incorrectly in one specific case which was not run into very often (when the credit to be given was larger than the max credit still allowed to be given on the invoice). Looking at past versions of the code it became quite apparent that the bug had been in place for over 6 years. I had needed to refactor and rename the (quite indecipherable) code to actually understand what it was doing before looking at correct data and comparing with the incorrect data to identify the bug.

## DST part deux
The integration test suite started failing in our CD pipeline. On investigation I found another DST bug - Joda Time advanced the clock by an hour on adding the number of minutes in 14 days (to now) since the 14 days ended after March 12th (when DST starts). The sql query doing the relevant filtering on the application side did not add the extra hour since it was just adding the 14 days in Oracle sql.


