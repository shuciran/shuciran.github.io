How to configure aws to get access to its functions:
```bash
❯ aws configure
AWS Access Key ID [****************6TDC]: AQLA5M37BDN6FJP76TDCAWS 
Secret Access Key [****************Fo1A]: OsK0o/glWwcjk2U3vVEowkvq5t4EiIreB+WdFo1A
Default region name [us-east-1]: us-east-1
Default output format [json]: json
```
Examples:
[[Epsilon#^2b8387]]

List all the functions within the lambda service running:

```bash
aws --endpoint-url=http://cloud.epsilon.htb lambda list-functions
{
    "Functions": [
        {
            "FunctionName": "**costume_shop_v1**",
            "FunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:costume_shop_v1",
            "Runtime": "python3.7",
            "Role": "arn:aws:iam::123456789012:role/service-role/dev",
            "Handler": "my-function.handler",
            "CodeSize": 478,
            "Description": "",
            "Timeout": 3,
            "LastModified": "2022-07-04T02:50:18.690+0000",
            "CodeSha256": "IoEBWYw6Ka2HfSTEAYEOSnERX7pq0IIVH5eHBBXEeSw=",
            "Version": "$LATEST",
            "VpcConfig": {},
            "TracingConfig": {
                "Mode": "PassThrough"
            },
            "RevisionId": "c23733b8-79c5-434a-ac38-2bdd1de4cc9e",
            "State": "Active",
            "LastUpdateStatus": "Successful",
            "PackageType": "Zip"
        }
    ]
}
```
Examples:
[[Epsilon#^426e39]]

After getting the functions then we can execute another command very useful to extract the info inside:

```bash
aws --endpoint-url=http://cloud.epsilon.htb lambda get-function --function-name=costume_shop_v1
{
    "Configuration": {
        "FunctionName": "costume_shop_v1",
        "FunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:costume_shop_v1",
        "Runtime": "python3.7",
        "Role": "arn:aws:iam::123456789012:role/service-role/dev",
        "Handler": "my-function.handler",
        "CodeSize": 478,
        "Description": "",
        "Timeout": 3,
        "LastModified": "2022-07-04T02:50:18.690+0000",
        "CodeSha256": "IoEBWYw6Ka2HfSTEAYEOSnERX7pq0IIVH5eHBBXEeSw=",
        "Version": "$LATEST",
        "VpcConfig": {},
        "TracingConfig": {
            "Mode": "PassThrough"
        },
        "RevisionId": "c23733b8-79c5-434a-ac38-2bdd1de4cc9e",
        "State": "Active",
        "LastUpdateStatus": "Successful",
        "PackageType": "Zip"
    },
    "Code": {
        **"Location": "<http://cloud.epsilon.htb/2015-03-31/functions/costume_shop_v1/code>"**
    },
    "Tags": {}
}
```

Examples:
[[Epsilon#^4d6122]]