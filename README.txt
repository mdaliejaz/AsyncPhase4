  ---------------------DISTRIBUTED POLICY EVALUATION----------------------

  This project implements the algorithm present in class - MVCC-TSO.
  History based policy evaluation is done using dist algo 
	(in a distributed and asynchronous manner)
  Also the latest version of Dist Algo is used (1.0.4) to deploy 
  coordinators, clients and master on different nodes.

  INSTRUCTIONS:

  To run the project run `sh Makefile` which will compile all the files
  To run with a test file FILE, run `dar master.da FILE`. For example to
  run the test case innerConflict.ini run 
  `dar master.da ../config/innerConflict.ini` from src/
  (Note: This assumes you have distalgo setup on your system)

  MAIN FILES:

  master.da: Reads the config.ini passed as an arguments and starts the
    required number of coordinators, database instance and clients.

  worker.da: During setup of each worker, the policy file is read and
    a hashmap of it (with key as action) is stored in memory.
    The worker also receives a request from co-ordinator and fires a query
    to the database to get attributes which are not present in cache.
    The most important functionality of the worker is to receive response
	from database and evaluate the policy.
	It does so by going through each rule associated with
	the particular action. It also sends updated attributes (with values)
	to the co-ordinator based on whatever update is defined in the policy rule
	matched. If no rule is matched, it returns false.
	If its a read only rule the worker directly send the result to client and
	calls the coordinator to update the timestamps of read attributes.

  client.da: During setup of a client the entire list of requests is
	generated and put in a queue. If its a pseudo random generator then
	a random function is called and a list of pseudo random requests
	are generated and put in the queue. 
	The client then serially pops from the queue and sends the request to
	the respective co-ordinator.
	The client also does static analysis of the policy and sends a bunch
	o details to the co-ordinator. These include defRead attributes, mightRead
	attributes and mightWrite attributes

  database.da: The database function initially reads an XML file (given in
	config.ini file). It receives requests from worker and fetches the
	attributes from the attributes. If an attribute
	is not present in db it writes to db with an empty value.
	The DB also receives request from co-ordinators to commit the cache
	into itself.

  coordinator.da: The main functionality of the coordinator is conflict
	resolution and prevent starvation. Versioning for every object and attribute is
	done by using a version Cache. Ther version cache hold a list of versions with
	rts, wts, pendingMightRead and value for each version of a particular attribute
	
	The co-ordinator receives requests from clients and it awaits on PCA / inserts into
	PCA the night Read Attrs for that object. PCA - potentially conflicting attributes
	used to prevent starvation. After this it updates the rts for def Read Attrs and
	pending Might Read for might read Atrrs. It then sends it to the other coordinator
	for valuation of the other object.
	This coordinator also does the similar thing and send the request to worker for evaluation.
	
	On receiving the Result from worker the coordinator checks for conflicts.
	Check 1: Is for outer conflicts on the defRead Attrs.
	Check 2: To Await all Pending Might Reads.
	If conflict detected in either case then restart
	If no conflict is detected send the result to client and update the read timestamps
	for attributes in might Read AttrX
	
  util.da: This class holds 4 class objects structures namely:- Request,
	Response, DBResponse and Policy Rule. Each class has its own __str__
    function (which was overridden) for enhanced logging.

  BUGS AND LIMITATIONS:

  All scenarios has been reproduced and tested using artificial delays and
  various policy rules. There were no bugs or limitations that we are aware of.

  CONTRIBUTIONS:

  Both of us participated in discussions and we both have complete knowledge of
  the entire code. In terms of coding (on a high level) :-

  Atanu: Co-ordinator class (for conflict resolution) and Master (Setup and teardown)
  Ali: Worker, database and client class (for static analysis)
  Common: Logging, utilities class, testing, code comments and pseudo code.

  OTHER COMMENTS:

  Stress testing and manual testing with various test cases including cases
  with attributes which start with $ have been done. Code is well documented
  and easy to understand. Modules for each functionality is seperate and there
  is no code duplication. Proper and consistent coding conventions followed throughout.

  REFERENCE:

  --> Dist Algo decumentation
  --> For logging: http://stackoverflow.com/questions/13649664/how-to-use-logging-with-pythons-fileconfig-and-configure-the-logfile-filename
