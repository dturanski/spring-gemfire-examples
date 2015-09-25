Gemfire Durable Client with Cache Listener Example:
=========================================================

Running the example:
==

* Clone the repo
* Open two terminal sessions:

In one:

     ./gradlew -q run-replicate-cs -Pmain=Server

In the other:

    ./gradlew -q run-replicate-cs -Pmain=Client
    
Press <ENTER> in the Server console to create additional entries.

Alternately you can import this project into an IDE and run from there, e.g.:

     ./gradlew eclipse
     
Import the individual project along with `spring-gemfire-examples-common`

Summary
==

This example is modified from the spring-gemfire-examples/basic/replicated-cs example to demonstrate unexpected behavior when configuring a durable client as documented. 

* Java version:   7.0.2 build 45714 01/28/2014 19:47:06 PST javac 1.6.0_26

* OS/X Yosemite

* spring-data-gemfire 1.4.2.RELEASE

### Expected behavior:

##### 1) Start Server and create two entries in a region

##### 2) Start client 

Configured with durable subscription, client cache, client region,registered interest in all keys, with CacheListener and readyForEvents.

Expect to fire cache entry events for the initial entries.

##### 3) Terminate client

Server should maintain durable subscription

##### 4) Create 2 more entries in Server

##### 5) Restart Client

Expect to fire cache entry events for newly created entries

### Actual behavior:


##### 1) Start Server:

Server creates 2 entries

###### server.log

    ...
    [info 2015/09/25 08:33:45.962 EDT  <main> tid=0x1] Initializing region Customer
    INFO [main] [LoggingCacheListener] - In region [Customer] created key [1] value [org.springframework.data.gemfire.examples.domain.Customer@1]
    INFO [main] [LoggingCacheListener] - In region [Customer] created key [2] value [org.springframework.data.gemfire.examples.domain.Customer@2]
    Press <Enter> to add  entries


##### 2) Start Client:

no cache entry events fired

###### client log


    ...
    [config 2015/09/25 08:34:04.740 EDT  <main> tid=0x1]        
    Startup Configuration:
    ### GemFire Properties defined with api ###
    durable-client-id=11
    durable-client-timeout=2000
    ...
    info 2015/09/25 08:34:04.741 EDT  <main> tid=0x1] Running in local mode since mcast-port was 0 and locators was empty.

    [info 2015/09/25 08:34:04.812 EDT  <Thread-0 StatSampler> tid=0xa] Disabling statistic archival.

    [config 2015/09/25 08:34:04.933 EDT  <poolTimer-gemfirePool-2> tid=0xe] Updating membership port.  Port changed from 0 to 52,147.

    [config 2015/09/25 08:34:05.065 EDT  <main> tid=0x1] Pool gemfirePool started with multiuser-authentication=false

    [info 2015/09/25 08:34:05.230 EDT  <main> tid=0x1] Sending ready for events to primary: Connection[localhost:40404]@1026871825
    Press <Enter> to terminate

    [info 2015/09/25 08:34:06.056 EDT  <Cache Client Updater Thread  on webster(57503):60485 port 40404> tid=0x12] Cache Client Updater Thread  on webster(57503):60485 port 40404 (localhost:40404) : ready to process messages.


Server creates a durable clue for client:

###### server log

    [info 2015/09/25 08:34:04.964 EDT  <ServerConnection on port 40404 Thread 1> tid=0x2e] ClientHealthMonitor: Registering client with member id identity(192.168.11.1(57510:loner):52147:55057f04,connection=1,durableAttributes=DurableClientAttributes[id=11; timeout=2000])

    [info 2015/09/25 08:34:05.039 EDT  <Handshaker 0.0.0.0/0.0.0.0:40404 Thread 1> tid=0x2d] Initializing region _gfe_durable_client_with_id_11_1_queue


##### 3) Terminate Client

Server detects client disconnect and retains durable queue
###### server log


    [info 2015/09/25 08:40:20.507 EDT  <ServerConnection on port 40404 Thread 2> tid=0x31] ClientHealthMonitor: Unregistering client with member id identity(192.168.11.1(57510:loner):52147:55057f04,connection=1,durableAttributes=DurableClientAttributes[id=11; timeout=2000])

    [info 2015/09/25 08:40:20.509 EDT  <ServerConnection on port 40404 Thread 2> tid=0x31] CacheClientNotifier: Keeping proxy for durable client named 11 for 2,000 seconds CacheClientProxy[identity(192.168.11.1(57510:loner):52147:55057f04,connection=1,durableAttributes=DurableClientAttributes[id=11; timeout=2000]); port=52150; primary=true; version=GFE 7.0.1].

    [info 2015/09/25 08:40:20.563 EDT  <Client Message Dispatcher for 192.168.11.1(57510:loner):52147:55057f04 (11)> tid=0x32] CacheClientProxy[identity(192.168.11.1(57510:loner):52147:55057f04,connection=1,durableAttributes=DurableClientAttributes[id=11; timeout=2000]); port=52150; primary=true; version=GFE 7.0.1] : Pausing processing


###### Note that if this step is skipped, the client is connected on step 4, the events are logged as expected:

    [info 2015/09/25 09:12:21.026 EDT  <Cache Client Updater Thread  on webster(58025):62365 port 40404> tid=0x13] Cache Client Updater Thread  on webster(58025):62365 port 40404 (localhost:40404) : ready to process messages.
    INFO [Cache Client Updater Thread  on webster(58025):62365 port 40404] [LoggingCacheListener] - In region [Customer] created key [3] value [org.springframework.data.gemfire.examples.domain.Customer@3]
    INFO [Cache Client Updater Thread  on webster(58025):62365 port 40404] [LoggingCacheListener] - In region [Customer] created key [4] value [org.springframework.data.gemfire.examples.domain.Customer@4]

##### 4) Add entries in Server:

Entries added

##### server log

    INFO [main] [LoggingCacheListener] - In region [Customer] created key [3] value [org.springframework.data.gemfire.examples.domain.Customer@3]
    INFO [main] [LoggingCacheListener] - In region [Customer] created key [4] value [org.springframework.data.gemfire.examples.domain.Customer@4]


##### 5) Restart Client

No cache entry events fired

###### client log

    [info 2015/09/25 08:42:27.280 EDT  <main> tid=0x1] Running in local mode since mcast-port was 0 and locators was empty.

    [info 2015/09/25 08:42:27.392 EDT  <Thread-0 StatSampler> tid=0xa] Disabling statistic archival.

    [config 2015/09/25 08:42:27.501 EDT  <poolTimer-gemfirePool-2> tid=0xe] Updating membership port.  Port changed from 0 to 52,190.

    [config 2015/09/25 08:42:27.593 EDT  <main> tid=0x1] Pool gemfirePool started with multiuser-authentication=false

    [info 2015/09/25 08:42:27.741 EDT  <main> tid=0x1] Sending ready for events to primary: Connection[localhost:40404]@1525409936
    Press <Enter> to terminate

    [info 2015/09/25 08:42:28.586 EDT  <Cache Client Updater Thread  on webster(57503):60485 port 40404> tid=0x11] Cache Client Updater Thread  on webster(57503):60485 port 40404 (localhost:40404) : ready to process messages.

Server detects client reconnect and resumes

###### server log

    [info 2015/09/25 08:42:27.510 EDT  <ServerConnection on port 40404 Thread 3> tid=0x35] ClientHealthMonitor: Registering client with member id identity(192.168.11.1(57612:loner):52190:62b08604,connection=1,durableAttributes=DurableClientAttributes[id=11; timeout=2000])

    [info 2015/09/25 08:42:27.580 EDT  <Handshaker 0.0.0.0/0.0.0.0:40404 Thread 2> tid=0x34] CacheClientProxy[identity(192.168.11.1(57612:loner):52190:62b08604,connection=1,durableAttributes=DurableClientAttributes[id=11; timeout=2000]); port=52193; primary=true; version=GFE 7.0.1]: Cancelling expiration task since the client has reconnected.

    [info 2015/09/25 08:42:27.763 EDT  <ServerConnection on port 40404 Thread 5> tid=0x37] CacheClientProxy[identity(192.168.11.1(57612:loner):52190:62b08604,connection=1,durableAttributes=DurableClientAttributes[id=11; timeout=2000]); port=52193; primary=true; version=GFE 7.0.1] : Resuming processing




