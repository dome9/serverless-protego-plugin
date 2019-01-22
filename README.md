PROTEGO PLUGIN FOR SERVERLESS 
===========  

**This plugin for serverless.com framework, will help you to secure your serverless application.**
- **Protego FSP**: allow to instrument each function with the Protego **FSP** agent.**
- **Protego Proact:** scan your functions before deployment, in order to detect:
    - Too Permissive Roles
    - Vulnerable Dependencies
    - Hardcoded Credentials 

  
### Pre requisite

- **package.json** file at the root of the serverless project
  > Note: `npm init` will create it automatically
- **Java 8*** (optional), required only when using Java function.
- **Docker***  (optional), required only for **Proact**


### Install
- From your serverless project directory, run:
- `$ npm install -D https://artifactory.app.protego.io/protego-serverless-plugin.tgz`
>:bulb: You can also use a specific version, (e.g `protego-serverless-plugin-1.0.0.tgz`) 
    

### Configure
1. Open Protego Console UI
2. Go to Account Settings / FSP Integrations and follow instructions:
3. **Set the plugin**: Under `plugins` section of the **serverless.yml** add :  
```yml
plugins: 
 ... 
 - serverless-protego-plugin 
```       

> :bulb: Note: The order of the plugin can change the behaviour! ensure that *serverless-protego-plugin* plugin will be the **last one** !!!


4. **Set Token and options**
>This configuration can be set under the **custom** section of the **serverless.yml** or using a **config file**.
  - **serverless.yml**:
    ```yml
    custom: 
     ...  
      protego:
        protegoAccountId: __PROTEGO_ACCOUNT_ID__
        protegoAccessToken: __PROTEGO_ACCESS_TOKEN__

        # Optional Args:
        
        fsp:
            # FSP instrument on/off
            Enabled:  <false | true>     # default is true
      
            # in order to not protect all functions you can use this flag
            protectAllFunctions: <false|true>     # default is true

            # In order to stop deployment on protection failure:
            exitOnFailure:       <false|true>     # default is false

            # In order to set a specific version, of the FSP agent:
            FSPVersion: 1.0.1                     # default is null
        
        proact:
            # Proact scanner on/off
            Enabled:  <false | true>     # default is false
            
            Features:
                PermissiveRole: 
                    Enabled:       <false|true>                     # default is true
                    FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
                
                VulnerableDependency: 
                    Enabled:       <false|true>                     # default is true
                    FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
                
                CredentialsUsage: 
                    Enabled:       <false|true>                     # default is true
                    FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
    ```

- **Config file**:

Another option is to use a json config file - `protego-config.json` under the root of the serverless project.

e.g:
```json
{
  "protegoAccountId"  : "__PROTEGO_ACCOUNT_ID__",
  "protegoAccessToken": "__PROTEGO_ACCESS_TOKEN__",
  "fsp" : {
      "exitOnFailure"     : true,
      "FSPVersion"        : "1.0.1"
  },
  "proact": {
    "Enabled": true
  }
}
```

> Note: you can combine both options , (* in case of duplicate values, **serverless.yml** will override **config** file)


### Configure each function (optional)
- In order to *enable* or *disable* protego FSP for a single function, add `protegoFsp: true (or false)` under each function block:
e.g.
```yml
  function_name:
  ...  
    protego:
        fsp:
            Enabled:       <false|true>                     # default is true
            
        proact:
            # Proact scanner on/off
            Enabled:  <false | true>     # default is false
            
            Features:
                PermissiveRole: 
                    Enabled:       <false|true>                     # default is true
                    FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
                
                VulnerableDependency: 
                    Enabled:       <false|true>                     # default is true
                    FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
                
                CredentialsUsage: 
                    Enabled:       <false|true>                     # default is true
                    FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
                
```

## Using the serverless Plugin

This plugin will act on `sls deploy` or `sls package` and also support single function flag `-f`

