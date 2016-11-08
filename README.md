# ABLUnitHandler
Currently ABLUnit tests procedures/classes can be executed either from command line,ANT build or _progres/prowin clients. To run the the same tests over HTTP and invoke them as RESTful services, we have taken advantage of PASOE as a platform to develop testing services as we have written a WebHandler to invoke ABLUnit tests and REST API's to manage them.


# Components and flow
![alt tag](https://cloud.githubusercontent.com/assets/4980960/20086979/7931dde8-a542-11e6-94dd-3d42ecfa1f81.png)

The above diagram illustrates the components involved while writing this functionality. 
 
## H2 HTTP Client
As the backend are RESTful Services, the HTTP Client can be any RESTful client(Javascript,curl etc). Also we can use tools like Jmeter or SOAPUI to run any kind of Performance tests with the same exposed URL's. Takes one or a list of tests as input  and passes each of them synchronously or asynchronously based on the requirement. 

## H2 ABLUnitHandler.cls
WebHandler responsible to take the input from HTTP Client, create the JSON input for ABLUnit and call "RunABLUnit" procedure synchronously in the same WEB transport or asynchronously on the APSV transport.

## H2 RunABLUnit.p
ABL procedure that invokes ABLUnit functions.

## H2 ABLUnitHandlerCore.cls
Core functionality that is been used to generate input, invoke ABLUnit and validate results.

## H2 GetTestSummary.cls
WebHandler that exposes REST API's to provide the summary and status of the tests ran.

## H2 TestMetrics.cls
All kinds of metrics information for the tests ran.

## H2 ABLUnitHandlerDB
OpenEdge Database to store the information of each test.

## H2 ABLUnit Results
ABLUnit results are stored in the deployed applications "/static/results" folder. The idea is to provide public access to the results and should be accessible based on test session's id.  
 
 
 

