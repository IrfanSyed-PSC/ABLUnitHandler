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


