# ABLUnitHandler
Currently ABLUnit tests procedures/classes can be executed either from command line,ANT build or _progres/prowin clients. To run the the same tests over HTTP and invoke them as RESTful services, we have taken advantage of PASOE as a platform to develop testing services as we have written a WebHandler to invoke ABLUnit tests and REST API's to manage them.


# Components and flow
![alt tag](https://cloud.githubusercontent.com/assets/4980960/20086979/7931dde8-a542-11e6-94dd-3d42ecfa1f81.png)

The above diagram illustrates the components involved while writing this functionality. 
 
##HTTP Client
As the backend are RESTful Services, the HTTP Client can be any RESTful client(Javascript,curl etc). Also we can use tools like Jmeter or SOAPUI to run any kind of Performance tests with the same exposed URL's. Takes one or a list of tests as input  and passes each of them synchronously or asynchronously based on the requirement. 

##ABLUnitHandler.cls
WebHandler responsible to take the input from HTTP Client, create the JSON input for ABLUnit and call "RunABLUnit" procedure synchronously in the same WEB transport or asynchronously on the APSV transport.

##RunABLUnit.p
ABL procedure that invokes ABLUnit functions.

##ABLUnitHandlerCore.cls
Core functionality that is been used to generate input, invoke ABLUnit and validate results.

##GetTestSummary.cls
WebHandler that exposes REST API's to provide the summary and status of the tests ran.

##TestMetrics.cls
All kinds of metrics information for the tests ran.

##ABLUnitHandlerDB
OpenEdge Database to store the information of each test.

##ABLUnit Results
ABLUnit results are stored in the deployed applications "/static/results" folder. The idea is to provide public access to the results and should be accessible based on test session's id.  
 
 
##Configuration
You need to have the below OpenEdge components to run ABLUnitHandler
PASOE(obviously, as you are running WebHandlers and running asynchronous requests)
OpenEdge Database

####Database Configuration
Create a copy of empty database and load ABLUnitHandlerDB.df to the newly created database. Start the database.
 
####PASOE Configuration
As it is an ABL Application, you can configure ABLUnitHandler in any OEABL(ROOT or any new OEABL) Application. Copy OpenEdge.zip to the PROPATH of your PASOE Instance and unzip it. Add PROPATH until the "OpenEdge" folder of the archive in openedge.properties file.

In the openedge.properties file, add the below handler configurations(or your can dump all the below handler configurations to a file and use oeprop command to merge them into openedge.properties file)
 
        handler1=OpenEdge.ABLUnitHandler.ABLUnitWebHandler: /auhandler/invoke
        handler2=OpenEdge.ABLUnitHandler.GetTestsSummary: /auhandler/sessions/all
        handler3=OpenEdge.ABLUnitHandler.GetTestsSummary: /auhandler/session/{sessionid}
        handler4=OpenEdge.ABLUnitHandler.GetTestsSummary: /auhandler/sessions/all/active
        handler5=OpenEdge.ABLUnitHandler.GetTestsSummary: /auhandler/sessions/all/complete
        handler6=OpenEdge.ABLUnitHandler.GetTestsSummary: /auhandler/sessions/all/aborted
        handler7=OpenEdge.ABLUnitHandler.GetTestsSummary: /auhandler/tests/{testcase}
        handler8=OpenEdge.ABLUnitHandler.GetTestsSummary: /auhandler/clientsession/{clientsessionid}
        handler9=OpenEdge.ABLUnitHandler.TestMetrics: /auhandler/metrics
 
Add the database details to the agentStartupParam of the PASOE Instance and you are done with the configuration.
 
####Testing ABLUnitHandler
As mentioned above, any HTTP client can be used to test the the ABLUnitHandler. For simplicity, we have used a javascript client so that both synchronous as well as asynchronous requests can be invoked. Please look into ABLUnitDriver.js on how the client is preparing the payload and invoking ABLUnitHandler.
 
If you want to control the asnychronous behavior from the client, then in ABLUnitHandler.cls call "RunABLUnit.p"  as below 

        RUN RunABLUnit.p(INPUT configJson,INPUT updatefile,INPUT sessguid,INPUT resultsDir,INPUT rec_id).

In-case, you do not want to control the asynchronous behavior and would like to leave it the WebHandler, then call "RunABLUnit.p"  as below 

       RUN RunABLUnit.p ON SERVER hAppSrv ASYNCHRONOUS SET asynchandle (INPUT configJson,INPUT updatefile,INPUT sessguid,INPUT resultsDir,INPUT rec_id).


In the attached client, we are running a list of tests as below
```javascript
{
  "tests": [
    "PASOEAPITesting.cls",
    "test1.p",
    "CallAPSV.p",
    "Communities_testing.p"
  ],
  "async": "true"
}
```


The list will be parsed and each client is run asynchronously as below.
```javascript
{
  "Sessionid": "<Client GUID> ",
  "ABLUnit": [
    {
      "tests": [
        {
          "test": "<testname>"
        }
      ]
    }
  ]
}
```

As each client will be running a list of tests, to keep those details unique for the user it is recommended to send a unique Sessionid for each request(refer to attached javascript client for example). This way we can query by the Client Session-id and look for its corresponding tests. If we do not send any Sessionid then in the database the clientguid value will be set to null and we have to query by each test session-id which will be returned on every POST request.
 
After successful execution of the test, you can query the status of the test by providing the Client GUID that was generated at the client as an input to the below API

        http://<hostname>:<port>/<ABLWebApp>/web/auhandler/clntsession/<Client GUID>
 
To access more details, look at the API's below


     
 
Description | Method
1.	
Invoke one or more ABLUnit tests
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/invoke | POST
2.	
Get all the list of tests executed
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/sessions/all
GET
3.	
Get all the list of the tests that have completed execution
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/sessions/all/complete
GET
4.	
Get all the list of the tests that are still running

http://<hostname>:<port>/<ABLWebApp>/web/auhandler/sessions/all/active
GET
5.	
Get all the list of the tests that are aborted after run

http://<hostname>:<port>/<ABLWebApp>/web/auhandler/sessions/all/aborted
GET
6.	
Get the summary of each client session
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/clientsession/<client session>
GET
7.	
Get the summary of each test session
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/session/<test session>
GET
8.	
Get the summary of all tests and sort them
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/sessions/all?sortby=<ClientSessions column name>
GET
9.	
Get the summary of all tests and filter them
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/sessions/all?filter=testStatus%3D%22PASSED%22
or
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/sessions/all?filter=testStatus%3D%22PASSED%22%26execTime>50
GET
10.	
Get the summary of all tests and apply both filter and sorting
 
http://<hostname>:<port>/ABLWebApp/web/auhandler/sessions/all?filter=testStatus=%22PASSED%22&Started%3E=2016-10-31T10:03:55.439-04:00&sortby=ExecTime
GET
11.	
Get the list of methods or procedures in a given ABLUnit test which has only @Test annotation
 
http://<hostname>:<port>/<ABLWebApp>/web/auhandler/tests/<testname>
GET
12.	
Get the ABLUnit result XML file
 
http://<hostname>:<port>/<ABLWebApp>/static/results/<sessionid>.xml
GET
 

