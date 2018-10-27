# tv-NodeJs

# 1) COD

# Folder Structure
	  |
	  |----client (This is the place for your application front-end files.)
	  |
	  |----common (this folder contains all models and their schemas)
	  |		|----models (this folder contains the models file where we write our api's)
	  |	
	  |----lib (this folder contains all the utilities method, third party methods)
	  |
	  |----node_modules
	  |
	  |----server
	  |     |----bin
	  |     |----boot (this is folder contains the authentication enable method)
	  |     |----views (this contains all the ejs views which are used in the project for the redirection from backend)
	  |     |----config.json (this file contains all the credentials and the controller)
	  |     |----datasources.json (This file contains the connection to the database)
	  |     |----middleware.json (this file contains all the type of validations which are used to call before any api call)
	  |     |----model-config.json (In this file we describe from which database the particular model will connect)
	  |     |----server.js 
	  |
	  |----package.json
	  |
	  |
  
# Technology & tools
 Nodejs, JavaScript, AWS S3 bucket, AWS Lambda function, Firebase Cloud Messaging(FCM), Loopback, Postgresql(Database), PostGis