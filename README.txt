  ---------------------DISTRIBUTED POLICY EVALUATION----------------------

  This project implments the algorithm present in paper <>.
  History based policy evaluation is done using dist algo 
	(in a distributed and asyncronous manner)

  INSTRUCTIONS:

  To run the project run `sh Makefile` which will compile all the files
  To run with a test file FILE, run `dar master.da FILE`. For example to
  run the test case subjectConflict.ini run 
  `dar master.da ../config/subjectConflict.ini` from src/
  (Note: This assumes you have distalgo setup on your system)

  MAIN FILES:

  master.da: Reads the config.ini passed as an arguments and starts the
    required number of cordinators, database instance and clients. It also
    gracefully kills all the processes when done.

  worker.da: During setup of each worker, the policy file is read and
    a hashmap of it (with key as action) is stored in memory.
    The worker also recieves a request from co-ordinator and fires a query
    to the database to get attributes which are not present in cache.
    The most important functionality of the worker is to receive response
	from database. It does so by going through each rule associated with
	the particular action. It also sends update attribute map to the
	co-ordinator based on whatever update is defined in the policy rule
	matched. If no rule is matched, it returns false.

  client.da: During setup of a client the entire list of requests is
	generated and put in a queue. If its a pseudo random generator then
	a random function is called and a list of pseudo random requests
	are generated and put in the queue. 
	The client then serially pops from the queue and sends the request to
	the respective subject co-ordinator.

  database.da: The database function initially reads an XML file (given in
	config.ini file). It receives requests from worker and fetches the
	subject and resource attributes from the attributes. If an attribute
	is not present in db it writes to db with an empty value.
	The DB also receives request from co-ordinators to commit the cache
	into itself. It does after waiting for a rand amount of time between
	min and max db latency (specified in config file)

  coordinator.da: The main functionality of the coordinator is conflict
	resolution and it is the interface for clients. The co-ordinator (subject
	instance) received requests from clients and enqueus it onto a 'request'
	queue. It then takes the subject tentative cache and sends the request
	to resource co-ordinator.The resource co-ordinator asks any of its
	workers (determined by round robin scheduling) to evaluate the policy.
	On receivng a reply from the worker, the subject co-ordinator checks
	for subject attribute conflicts. If there is no conflicts it updates its
	tentative cache and send the request to resource co-ordinator for resource
	conflict check. If there was no resource conflict the tentative cache
	is committed to main cache. In either case if there is a conflict the tentative
	cache is reverted and the request restarted. Ordering of requests is done
	by storing them in a response queue.

  util.da: This class holds 4 class objects structures namely:- Request,
	Response, DBResponse and Policy Rule. Each class has its own __str__
    function (which was overridden) for enhanced logging.

  BUGS AND LIMITATIONS:

  All scenarios has been reprodued and tested using artifical delays and
  various plicy rules. There were no bugs or limitations that we are aware of.

  CONTRIBUTIONS:

  Both of us participated in discussions and we both have complete knowledge of
  the entire code. In terms of coding (on a high level) :-

  Ali: Co-ordinator class (for conflict resolution) and Master (Setup and teardown)
  Atanu: Worker and database class (for policy evaluation) and Client class
  Common: Logging, utilities class, testing, code comments and pseudo code.

  OTHER COMMENTS:

  Stress testing and manual testing with various test cases including cases
  with attributes which start with $ have been done. Code is well documented
  and easy to understand. Modules for each functionality is seperate and there
  is no code duplication. Proper and consistent coding conventions follow throughout.

  REFERENCE:

  --> Dist Algo decumentation
  --> For logging: http://stackoverflow.com/questions/13649664/how-to-use-logging-with-pythons-fileconfig-and-configure-the-logfile-filename
