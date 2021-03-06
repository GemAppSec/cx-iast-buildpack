# Cx IAST Cloud Foundry Buildpack Support

This buildpack provides CxIAST Agent Instrumentation for Java applictions running on Cloud Foundry. It is designed to be used with the official Cloud Foundry Java Buildpack in a multi buildpack approach.

# Usage

## User provided service
Create a user provided service named ```checkmarx``` and bind it to your application. In its credentials specify the ```iast_server``` key pointing to your CxIAST server. The buildpack will download the agent from this server.

For example:
```
{
 "VCAP_SERVICES": {
  "user-provided": [
   {
    "binding_name": "",
    "credentials": {
     "iast_server": "https://YOUR-CXIAST-SERVER:YOUR-PORT"
    },
    "instance_name": "checkmarx",
    "label": "user-provided",
    "name": "checkmarx",
    "syslog_drain_url": "",
    "tags": [],
    "volume_mounts": []
   }
  ]
 }
}
```
## Deployment
### Deploy with a manifest file
Create a ```manifest.yml``` file with content similar to this and specify the buildpacks in this order. Then launch your application with ```cf push```. Bulid pack order is important because ```java_buildpack``` acts as the final buildpack.
```
# manifest.yml
---
applications:
- name: YOUR-APP
  memory: 1G
  instances: 1
  path: ./target/cloudfoundry-demo-0.0.1-SNAPSHOT.jar  
  buildpacks:
    - https://github.com/checkmarx-ts/cx-iast-buildpack.git
    - java_buildpack   
  timeout: 180
  ```

### Deploy with cf push
Specify multiple build packs on the command line like this:
```cf push YOUR-APP -b https://github.com/checkmarx-ts/cx-iast-buildpack.git -b java_buildpack```

### Agent activation & Buildpack detection
Currently, the agent is always active and the Buildpack will always perform Java instrumentation whenever it is used. To control the agent activation by env you should change the Buildpack specification with your existing tooling.


# Configuration
## cxAppTag
The default ```cxAppTag``` value is the application's name in Cloud Foundry. Override this by setting a ```cxAppTag``` environment variable for the application in Cloud Foundry.

## cxTeam
The default team is ```CxServer```. Override this by setting a ```cxTeam``` environment variable. The team must exist on the CxIAST Server - it will not be created automatically.

## Timeout
If the app doesn't start fast enough it may be due to initial instrumentation. I recommened setting your timeout to 180 seconds when using this buidpack. 

# Logging
The ‘’’configuration/checkmarx-logback.xml’’’ is a modified log configuration from the standard agent. It enables the STDOUT appender at the INFO level so logs will be picked up by the Cloud Foundry Loggregator and be seen by ```cf logs``` command and be included in any log drains. 

# References
* https://github.com/cloudfoundry/java-buildpack/blob/master/docs/framework-multi_buildpack.md
* https://docs.cloudfoundry.org/buildpacks/use-multiple-buildpacks.html
* https://docs.cloudfoundry.org/buildpacks/developing-buildpacks.html
