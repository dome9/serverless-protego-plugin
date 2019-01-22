PROTEGO - CLI TOOL 
===========  

**Protego CLI tool, will help you secure your serverless application.**

The tool has 2 main features:
- **[Protego FSP](#fsp-function-self-protection)**: Instrument Lambda function with the Protego **FSP (Function Self Protection)** agent
- **[Protego Proact](#proact-function-scanner):** scan your functions before deployment, in order to detect:
    - Permissive Roles
    - Vulnerable Dependencies
    - Hardcoded Credentials 
    
---   

### Prerequisite

- **Nodejs 6.10** 
- **Java 8*** (optional), only when instrumenting FSP into a Java function.
- **Docker*** (optional), only for **Proact**


---


### Install
- `$ npm install -g https://artifactory.app.protego.io/protego-serverless-plugin.tgz`
>:bulb: You can also use a specific version, (e.g `protego-serverless-plugin-1.0.0.tgz`) 

---

## General
```bash
Usage: protego [options] [command]

Options:

  -V, --version     output the version number
  -h, --help        output usage information

Commands:

  fsp [options]     protect a given function
  proact [options]  scan Lambda functions

```

---
    
## FSP (Function Self Protection)

Function Self Protection is a security layer that is embedded around or in the function. Function self-protection enables you to block attacks.

FSP is able to dynamically inspect various points within the flow of functions using various mechanisms such as pattern matching, flow analysis, blacklisting, whitelisting, and apply policies such as reporting, blocking in response to suspicious activity.

### USAGE

## **$ protego fsp** (command line)

This command instrument a given lambda .zip code file, and output a new protect zip file, ready to be uploaded to AWS account.

The input is the location of the zip file, the handler name and the runtime.

This tool require also the  Protego Account ID & Protego Token (you can find protego Account ID & Protego Token in Protego's account integration page ) 
  
  
```
Usage: protego fsp [options]

protect a given function

Options:

  -i, --input <path>                  path to input code package file (zip/jar)
  -H, --handler <handler>             handler path as configured in AWS Lambda
  -a, --aws-account-id <id>           AWS Account ID
  -p, --protego-account-id <id>       Protego Account ID
  -t, --protego-access-token <token>  Protego Token
  -r, --runtime <runtime>             Environment runtime [python2.7, python3.6, python3.7, java8, nodejs8.10, nodejs6.10, dotnetcore2.0, dotnetcore2.1]
  -f, --fsp-version <version>         (optional) FSP Version (default: latest)
  -o, --output [path]                 (optional) path to protected output package file (zip/jar)
  -q, --quite                         (optional) quite mode: output only the json output
  -v, --verbose                       (optional) verbose debug logs
  -h, --help                          output usage information
```


## **Config file**:

You can pass part of the above args via a config file named : `protego-config.json`.

> This file have to be located on the same directory that you run this command.

e.g:
```json
{
  "protegoAccountId": "__PROTEGO_ACCOUNT_ID__",
  "protegoAccessToken": "__PROTEGO_ACCESS_TOKEN__",
  "FSPVersion": "1.0.1"
}
```


## Deployment:

Upload the protected zip to your AWS account.

>:bulb: The tool will output an environment variable that you have to add to the Lambda Function.

e.g
```json
{ "PROTEGO_FSP_CUSTOMER_ACCOUNT_ID": "161635420639:c45e26a304be5b9d6757dfdf946aeeda" }
```



## Example:

```bash
protego fsp \
    --input ../../_build/my_function.zip \
    --handler handler_file.lambda_handler_method \
    --aws-account-id 123123123123 \
    --protego-account-id xxxx-xxxx-xxxx-xxxx
    --protegoAccessToken "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX="
    --runtime python2.7
```


---
---

## Proact (Function Scanner)

scan your functions before deployment, in order to detect:
 - Too Permissive Roles
 - Vulnerable Dependencies
 - Hardcoded Credentials 
    
### USAGE

## **$ protego proact** (command line)
This command get as input a [`protego.yml`](#protegoyml) file describing functions list, and scan these functions.
The output results can be found under [`protego_output`](#output) folder.

> When scan failed, the cli will exit with error code non equal to zero

```
Usage: protego proact [options]

scan Lambda functions

Options:

  -i, --input [path]                  path to protego.yml input file (default: protego.yml)
  -a, --aws-account-id <id>           AWS Account ID
  -p, --protego-account-id <id>       Protego Account ID
  -t, --protego-access-token <token>  Protego Token
  -o, --output [path]                 (optional) folder path of reports outputs (default: protego_output)
  --proact-version <version>          (optional) Proact Version (default: latest)
  --tag [tag]                         (optional) tags key=value (default: )
  -f, --function [function name]      (optional) function/s e.g func1, func2 (default: )
  -s, --store-report                  (optional) store report history on protego backend
  -l, --parallel [num]                (experimental) number of parallel functions to scan
  -q, --quite                         (optional) quite mode: output only the json output
  -v, --verbose                       (optional) verbose debug logs
  -h, --help                          output usage information

```

**e.g:**

```bash
protego proact \
    --aws-account-id 123123123123 \
    --input protego.yml
     
```

> Note: First running of the tool will install dependencies and may take a few minutes.

## Input

**e.g folder structure**:
```
.
├── protego.yml
├── cloud-formation-template.yml
..
│ 
├─ functions    
│  ├── e.g ExampleLambdaFunction
│  │   └── handler_file.py
│  └-─ e.g my_function1
│      └── handler_file.py
..
│
└── protego_output
    ├── LambdaFunction1.yaml
    ├── LambdaFunction2.yaml
    └── ProtegoScanResults.yaml

```

### protego.yml

```yaml
Account:
  ProtegoAccountId:   ___ProtegoAccountId___        # required (or use --protego-account-id arg)
  ProtegoAccessToken: ___ProtegoAccessToken___      # required (or use --protego-access-token arg)
  StoreJobReport:     <true|false>                  # optional, default: false
  Tags:               <key, value Dictonary>        # optional, default: empty
    A_KEY: A_VALUE
    B_KEY: B_VALUE
    ..

Globals:
  
  ControlPoints:
    PermissiveRole: 
      Enabled:       <false|true>                     # default is true
      FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
      
    VulnerableDependency: 
      Enabled:       <false|true>                     # default is true
      FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
       
    CredentialsUsage: 
      Enabled:       <false|true>                     # default is true
      FailThreshold: <None|Low|Medium|High|Critical>  # default is Low
  
  # Optional Mode
  ProtegoGeneratedRole: <true|false>                  # optional, in this mode it will only generate roles, without scanning 
 
Integrations:                                         # optional
  Cloudformation:
    Template: cloud-formation-template.yml            # the template path  
    Parameters:                                       # optional CF, params (key, value)
      param1: value1

      
Functions:
  # e.g with cloud formation template (only code location is require) 
  ExampleLambdaFunction:
    CodeLocation: functions/ExampleLambdaFunction
    
    ControlPoints: <as defined above in globals>      # optional
    
  # e.g full details, (without cloud formation template)
  MyLambdaFunction1:
    Runtime : python2.7 | node6.10 | node8.10                               
    Handler: handler.lambda_handler
    CodeLocation: functions/my_function1
    EventSourceMappings:
      - EventSourceArn: arn:aws:kinesis:us-east-1:123456789012:stream/RecipientStream
      - EventSourceArn: arn:aws:dynamodb:us-east-1:123456789012:table/aaaa/stream/2017-11
    DeadLetterConfig:
      TargetArn: arn:aws:sns:us-east-1:373507998356:func_name
    TracingConfig:
      Mode: Active
    Role: arn:aws:iam::123456789012:role/MyRole

  # e.g how to skip scanning a function
  someOtherFunction:
    Runtime : nodejs6.10
    Handler: handler.lambda_handler
    CodeLocation: path_to_the_code
    Enabled: false

```


## Output:

**ProtegoScanResults.yaml**

**e.g** 
```yaml
Failed:
  Functions: []
  Total: 0
Skipped:
  Functions: 
  - someOtherFunction
  Total: 1
Successful:
  Functions:
  - ExampleLambdaFunction
  - MyLambdaFunction1
  Total: 2
```
  
  
a .yml file will be genrated for each function:

e.g:

```yaml
CredentialsUsage: {}

PermissiveRole:
  RedundantPermissions:
    '*':
    - '*'
    dynamodb:
    - dynamodb:UpdateItem
    sqs:
    - sqs:SendMessage
    - sqs:SendMessageBatch
    - sqs:GetQueueUrl
    sts:
    - sts:AssumeRole
    
  SuggestedRole:
    Statement:
    - Action:
      - dynamodb:PutItem
      Effect: Allow
      Resource:
      - '*'
      Sid: ProtegoGenerated0862057e
    - Action:
      - logs:CreateLogGroup
      - logs:CreateLogStream
      - logs:PutLogEvents
      Effect: Allow
      Resource:
      - '*'
      Sid: ProtegoGeneratedbfb0047a
    - Action:
      - sns:Publish
      Effect: Allow
      Resource:
      - '*'
      Sid: ProtegoGenerateda1902c2b

VulnerableDependency:
  VulnerableDepsItems:
  - cvss-score: '6.8'
    description: Cross-site request forgery in the REST API in IPython 2 and 3.
    name: ipython
    version: 3.2.2
```
  
