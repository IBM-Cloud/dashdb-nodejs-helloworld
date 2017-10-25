# Db2 Warehouse on Cloud -  Node.js Hello World Sample

The code in this repository provides a simple implementation of an app running on Node.js runtime demonstrating how to connect Node.js applications to the IBM Db2 Warehouse on Cloud service.

You can bind a IBM Db2 Warehouse on Cloud service instance to an app running on Node.js runtime in the IBM Cloud (Bluemix) and then work with the data in the Db2 database (relational SQL database). The Bluemix Node.js runtime will automatically lay down native driver dependencies when you have a Db2 Service instance bound to your app. The sample illustrated here uses express and jade node modules.

For issues that you encounter with this service, go to [**Get help**](https://developer.ibm.com/bluemix/) in the Bluemix developer community.

[![Deploy to Bluemix](https://bluemix.net/deploy/button.png)](https://bluemix.net/deploy)

![Bluemix Deployments](https://deployment-tracker.mybluemix.net/stats/6dc62259729f4ccc57f616d7f1e7315b/badge.svg)

## Running the app on Bluemix

1. If you do not already have a Bluemix account, [sign up here](https://console.ng.bluemix.net/registration/)

2. Download and install the [Cloud Foundry CLI](https://github.com/cloudfoundry/cli/releases) tool

3. Clone the app to your local environment from your terminal using the following command:

  ```
  git clone https://github.com/IBM-Bluemix/dashdb-nodejs-helloworld.git
  ```

4. `cd` into this newly created directory

5. Open the `manifest.yml` file and change the `host` value to something unique.

  The host you choose will determinate the subdomain of your application's URL:  `<host>.mybluemix.net`

6. Connect to Bluemix in the command line tool and follow the prompts to log in.

  ```
  $ cf api https://api.ng.bluemix.net
  $ cf login
  ```

7. Create a Db2 Warehouse on Cloud service instance in Bluemix.

  ```
  $ cf create-service dashDB Entry dashDB-nodesample
  ```

8. Push the app to Bluemix.

  ```
  $ cf push
  ```

And voila! You now have your very own instance of an app running Node.js runtime and connecting and querying the dashDB service on Bluemix.

## Run the app locally
1. If you do not already have a Bluemix account, [sign up here](https://console.ng.bluemix.net/registration/)

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

6. Create a dashDB service using your Bluemix account and replace the corresponding credentials in your app.js file

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


## Decomposition Instructions

The primary purpose of this demo is to provide a sample implementation of an app running on Node.js runtime demonstrating how to connect Node.js applications to Db2 Warehouse on Cloud service on Bluemix. The relevant code for this integration is located within the `app.js` file.

The following components are required to connect Db2 from a Node.js application. The are all described in further detail in this topic.

- package.json
- Node.js program
- A dashDB service instance

#### package.json
The package.json contains information about your app and its dependencies. It is used by npm tool to install, update, remove and manage the node modules you need. Add the ibm_db dependency to your package.json:
```
{
  "engines": {
    "node": "0.10.33"
  },
  "name": "dashdbnodesample",
  "version": "0.0.1",
  "description": "A sample node app for connecting to dashDB service on Bluemix",
  "dependencies": {
    "cf-deployment-tracker-client": "0.0.8",
    "express": "3.5.x",
    "ibm_db": "0.0.19",
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

For more information on the structure of the VCAP_SERVICES environment variable see [Getting Started with dashDB Service](http://www.ng.bluemix.net/docs/#services/dashDB/index.html#dashDB)

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
				conn.query("SELECT FIRST_NAME, LAST_NAME, EMAIL, WORK_PHONE from GOSALESHR.employee FETCH FIRST 10 ROWS ONLY", function(err, tables, moreResultSets) {

							
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

The primary source of debugging information for your Bluemix app is the logs. To see them, run the following command using the Cloud Foundry CLI:

  ```
  $ cf logs dashdbnodesample --recent
  ```
For more detailed information on troubleshooting your application, see the [Troubleshooting section](https://www.ng.bluemix.net/docs/troubleshoot/tr.html) in the Bluemix documentation.

## Contribute
We are more than happy to accept external contributions to this project, be it in the form of issues or pull requests. If you find a bug or want an enhancement to be added to the sample application, please report via the [issues section](https://github.com/IBM-Bluemix/dashdb-nodejs-helloworld/issues) or even better, fork the project and submit a pull request with your fix. Pull requests will be evaulated on an individual basis based on value add to the sample application.


## Privacy Notice
The dashdb-nodejs-helloworld sample web application includes code to track deployments to Bluemix and other Cloud Foundry platforms. The following information is sent to a [Deployment Tracker](https://github.com/cloudant-labs/deployment-tracker) service on each deployment:

* Application Name (application_name)
* Space ID (space_id)
* Application Version (application_version)
* Application URIs (application_uris)

This data is collected from the VCAP_APPLICATION environment variable in IBM Bluemix and other Cloud Foundry platforms. This data is used by IBM to track metrics around deployments of sample applications to IBM Bluemix. Only deployments of sample applications that include code to ping the Deployment Tracker service will be tracked.

### Disabling Deployment Tracking

Deployment tracking can be disabled by removing `require("cf-deployment-tracker-client").track();` from the beginning of the `app.js` file.

### Related Links
- [IBM DB2 node.js API](https://www.npmjs.org/package/ibm_db#api)

- [Using IBM Db2 from node.js](https://www.ibm.com/developerworks/community/blogs/pd/entry/using_ibm_db2_from_node_js4?lang=en)

- [IBM Db2 v10.5 Knowledge Center](https://www-01.ibm.com/support/knowledgecenter/SSEPGG_10.5.0/com.ibm.db2.luw.kc.doc/welcome.html)
- [IBM Db2 developerWorks](http://www.ibm.com/developerworks/data/products/db2/)
