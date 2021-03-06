# Validating PODs

PODs(physical nodes) are deployed everynight using Jenkins Build System.  After a successful
installation of the POD, test jobs are triggered which validate the following
categories of tests

* Post Installation Verification

* Functional Tests

## Post Installation Tests

These tests perform the following validations

* Required services are running
* All deployments are successfully rolled out and matches replicas to available replicas
* All pods are running
* Pods have external connectivity
* Pods can ping the kube-system namespace
* Nodes are healthy (checking “ready”, “outofdisk”, “memorypressure”, “diskpressure”)
* Required containers are in running state

To execute the test, perform the following from the client machine

```bash
cd cord-tester/src/test/diag
pybot SanityK8Pod.robot
```

## Functional Tests

Control tests can be executed on the POD once the
sanity checks are successful.

### Executing Control Plane Tests

To validate the end-end functionality checks on the RCORD Lite APIs, the
following control plane test can be executed.

* Edit the attributes shown below in the properties file

```bash
$ cd  cord-tester/src/test/cord-api/Properties
$ cat RestApiProperties.py

SERVER_IP = 'localhost'
SERVER_PORT = '9101'
USER = 'xosadmin@opencord.org'
PASSWD = ''
```

* To run the tests

```bash
cd cord-tester/src/test/cord-api/Tests/
pybot VOLTDevice_Test.txt
pybot RCORDLite_E2ETest.txt 
```

### Data Plane Tests

Data plane tests include validations of both top-down and zero-touch approaches as well as workflow validations. Follow the guide above to edit the properties file before running the tests.

**Validating top-down and zero-touch approaches**

`Subscriber_StatusChecks.txt` validates subscriber status and end-to-end ping. Execute the following commands to run the test.
```bash
cd cord-tester/src/test/cord-api/Tests/
pybot -v init_state:disabled -v INITIAL_STATUS:FAIL -v ENABLE_STATUS:PASS -e zerotouch Subscriber_StatusChecks.txt
```
Remember to change the values (ip, gateway, username, password, etc.) in the `Variables` section of the script to point the test to the source and destination hosts in your environment before running it.

Similarly, run the follownig commands to validate zero-touch approach. Note that the arguments passed to the test script are different from the top-down approach.
```bash
cd cord-tester/src/test/cord-api/Tests/
pybot -v init_state:awaiting-auth -v INITIAL_STATUS:FAIL -v ENABLE_STATUS:FAIL -v MACIP_STATUS:PASS Subscriber_StatusChecks.txt
```

**Validating AT&T workflow**

Test scripts and input data for validating AT&T workflow are under `cord-tester/src/test/cord-api/Tests/WorkflowValidations`. The same test script e.g. `ATT_Test001.robot` works with different POD setups. Instead of hardcoding the POD specific variables in the test script, it relies on a separated configuration file which describes POD setup. To create a configuratino file for your POD please take a look at [this example](https://github.com/opencord/pod-configs/blob/master/deployment-configs/flex-pod1-olt.yaml).

Input data are stored under `cord-tester/src/test/cord-api/Tests/WorkflowValidations/data/`. Please create a new folder with the name of your POD and copy and data files from e.g. `flex-pod1-olt` folder and edit them with the correct values on your POD.

Also make sure that the variables in the test script (e.g. `ATT_Test001.robot`) are correct. Specifically, verify that `${POD_NAME}` is the same as the folder name you created above, and `${KUBERNETES_CONF}` is pointing to your Kubernetes configuration file.

After updating all these POD specific values, execute the following commands to trigger the test

```bash
cd cord-tester/src/test/cord-api/Tests/WorkflowValidations
pybot -V PATH_TO_YOUR_POD_CONFIGURATION_FILE ATT_Test001.robot
 ```

Note that `PATH_TO_YOUR_POD_CONFIGURATION_FILE` should point to the yaml file that describes your POD setup (see above).
