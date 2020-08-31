# Db2 on Cloud -  Node.js Hello World Sample

The code in this repository provides a simple implementation of an app running on Node.js runtime demonstrating how to connect Node.js applications to the IBM Db2  on Cloud service.

You can bind a IBM Db2 on Cloud service instance to an app running on Node.js runtime in the IBM Cloud and then work with the data in the Db2 database (relational SQL database). The Node.js runtime will automatically lay down native driver dependencies when you have a Db2 Service instance bound to your app. The sample illustrated here uses express and jade node modules.

## Running the app on Bluemix

1. If you do not already have a IBM Cloud account, [sign up here](https://cloud.ibm.com/registration)

2. Download and install the [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cli-install-ibmcloud-cli) tool

3. Clone the app to your local environment from your terminal using the following command:

  ```
  git clone https://github.com/IBM-Bluemix/dashdb-nodejs-helloworld.git
  ```

4. `cd` into this newly created directory

5. Open the `manifest.yml` file and change the `host` value to something unique.

  The host you choose will determine the subdomain of your application's URL:  `<host>[.region].mybluemix.net`

6. Connect to IBM Cloud in the command line tool and follow the prompts to log in.
   You can find the most appropriate API endpoint for running your application at [Cloud Foundry](https://cloud.ibm.com/docs/cloud-foundry-public?topic=cloud-foundry-public-endpoints) - the following example assume a USA-based deployment.

  ```
  $ ibmcloud cf api  https://api.us-south.cf.cloud.ibm.com  # USA
  $ ibmcloud cf login
  $ ibmcloud target --cf
  ```

7. Create a Db2 on Cloud service instance in IBM Cloud.

  ```
  $ ibmcloud cf create-service "dashDB For Transactions" Lite dashDB-nodesample
  ```

8. Push the app to Cloud Foundry on IBM Cloud.

  ```
  $ ibmcloud cf push
  ```

And voila! You now have your very own instance of an app running Node.js runtime and connecting and querying the dashDB service on Bluemix.

## Run the app locally
1. If you do not already have an IBM Cloud account, [sign up here](https://cloud.ibm.com/registration)

2. If you have not already, [download node.js](https://nodejs.org/en/download/) and install it on your local machine.

3. Clone the app to your local environment from your terminal using the following command:

  ```
    git clone https://github.com/IBM-Bluemix/dashdb-nodejs-helloworld.git
  ```

4. `cd` into this newly created directory

5. Install the required npm and bower packages using the following command

  ```
  npm install
  ```

6. Create a Db2 on Cloud service [Lite plan](https://cloud.ibm.com/catalog/services/db2) using your IBM Cloud account and replace the corresponding credentials in your app.js file

```
        db: "BLUDB",
        hostname: "xxxx",
        port: 50000,
        username: "xxx",
        password: "xxx"
```

7. Start your app locally with the following command

  ```
  node app.js
  ```

This command will start your application. When your app has started, your console will print that your `Express server listening on port 3000`.


##Instructions

The primary purpose of this demo is to provide a sample implementation of an app running on Node.js runtime demonstrating how to connect Node.js applications to Db2 Warehouse on Cloud service on Bluemix. The relevant code for this integration is located within the `app.js` file.

The following components are required to connect Db2 from a Node.js application. The are all described in further detail in this topic.

- package.json
- Node.js program
- A Db2 on Cloud (dashDB for Transactions) Lite service instance

#### package.json
The package.json contains information about your app and its dependencies. It is used by npm tool to install, update, remove and manage the node modules you need. Add the ibm_db dependency to your package.json:
```
{
  "engines": {
    "node": ">=0.10.0"
  },
  "name": "dashdbnodesample",
  "version": "0.0.1",
  "description": "A sample node app for connecting to dashDB service on Bluemix",
  "dependencies": {
    "cf-deployment-tracker-client": "0.0.8",
    "express": "3.5.x",
    "ibm_db": ">=2.0.0",
    "jade": "1.11.0"
  },
  "scripts": {
    "start": "node app.js"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/IBM-Bluemix/dashdb-nodejs-helloworld.git"
  }
}

```

### Connecting to Db2 from Node.js code

In your Node.js code, parse the VCAP_SERVICES environment variable to retrieve the database connection information and connect to the database as shown in the following example.

For more information on the structure of the VCAP_SERVICES environment variable, see [Getting Started with Db2 Service](https://cloud.ibm.com/docs/Db2onCloud?topic=Db2onCloud-getting-started)

```
var app = express();

// all environments
app.set('port', process.env.PORT || 3000);
...
var db2;

var env = JSON.parse(process.env.VCAP_SERVICES);
db2 = env['dashDB'][0].credentials;

var connString = "DRIVER={DB2};DATABASE=" + db2.db + ";UID=" + db2.username + ";PWD=" + db2.password + ";HOSTNAME=" + db2.hostname + ";port=" + db2.port;

app.get('/', routes.listSysTables(ibmdb,connString));
```

In the routes/index.js file, open the connection using the connection string and query the database as follows.
```
exports.listSysTables = function(ibmdb,connString) {
	return function(req, res) {
		   
	   ibmdb.open(connString, function(err, conn) {
			if (err ) {
			 res.send("error occurred " + err.message);
			}
			else {
				conn.query(
				"SELECT 
					schemaname as schema_name,
    					owner as schema_owner,
    					case ownertype 
        					when 'S' then 'SYSTEM'
        					when 'U' then 'USER' 
    					end as schema_owner_type,
    					definer as schema_definer,
    					case definertype 
        					when 'S' then 'SYSTEM'
        					when 'U' then 'USER'
    					end as schema_definer_type
				FROM SYSCAT.SCHEMATA
				ORDER by schema_name", 
				function(err, tables, moreResultSets) {		
				if ( !err ) { 
					res.render('tablelist', {
						"tablelist" : tables
					 });					
				} else {
				   res.send("error occurred " + err.message);
				}

				/*
					Close the connection to the database
					param 1: The callback function to execute on completion of close function.
				*/
				conn.close(function(){
					console.log("Connection Closed");
					});
				});
			}
		} );
	   
	}
}
```

## Troubleshooting

The primary source of debugging information for your IBM Cloud app is the logs. To see them, run the following command using the Cloud Foundry CLI:

  ```
  $ ibmcloud cf logs dashdbnodesample --recent
  ```
For more detailed information on troubleshooting your application, 
see the [Troubleshooting section](https://cloud.ibm.com/docs/cloud-foundry-public?topic=cloud-foundry-public-runtimes) in the IBM Cloud documentation.

## Contribute
We are more than happy to accept external contributions to this project, be it in the form of issues or pull requests. If you find a bug or want an enhancement to be added to the sample application, please report via the [issues section](https://github.com/IBM-Bluemix/dashdb-nodejs-helloworld/issues) or even better, fork the project and submit a pull request with your fix. Pull requests will be evaulated on an individual basis based on value add to the sample application.


## Privacy Notice
The dashdb-nodejs-helloworld sample web application includes code to track deployments to IBM Cloud and other Cloud Foundry platforms. The following information is sent to a [Deployment Tracker](https://github.com/cloudant-labs/deployment-tracker) service on each deployment:

* Application Name (application_name)
* Space ID (space_id)
* Application Version (application_version)
* Application URIs (application_uris)

This data is collected from the VCAP_APPLICATION environment variable in IBM Cloud and other Cloud Foundry platforms. This data is used by IBM to track metrics around deployments of sample applications to IBM Bluemix. Only deployments of sample applications that include code to ping the Deployment Tracker service will be tracked.

### Disabling Deployment Tracking

Deployment tracking can be disabled by removing `require("cf-deployment-tracker-client").track();` from the beginning of the `app.js` file.

### Related Links
- [IBM DB2 node.js API](https://www.npmjs.org/package/ibm_db#api)

- [Using IBM Db2 from node.js](https://github.com/ibmdb/node-ibm_db/)

- [IBM Db2 Knowledge Center](https://www-01.ibm.com/support/knowledgecenter/SSEPGG_10.5.0)
- [IBM Db2 community](https://community.ibm.com/community/user/hybriddatamanagement/communities/db2-home)
