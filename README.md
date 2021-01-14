# weekly23_aws

## CloudFormation

### Template

```yml
---
Description: Example CloudFormation Template
Parameters:
  Subnet:
    Description: Where to put this instance
    Type: AWS::EC2::Subnet::Id
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup::Id
  InstanceType:
    Type: String
    Default: t2.nano
    AllowedValues:
      - t2.nano
      - t2.micro
Resources:
  SimpleDoersAWSInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0a91cd140a1fc148a
      InstanceType:
        Ref: InstanceType 
      SecurityGroupIds:
        - Ref: SecurityGroup 
      SubnetId:
        Ref: Subnet
      UserData:
        Fn::Base64: |
          #!/bin/bash -xe
          sudo apt install -y nginx
          sudo service nginx start
```

### Change Set

### AWS Lambda S3 to create resouce

- IAM console
- Roles
- Create a Role
- Lambda
- Next Permissions
- Select [x] AmazonS3FullAccess
- Select Tags -> Review
- Name: LambdaW3FullAccessRole
- Create Role
- Role ARN: arn:aws:iam::620157586684:role/LambdaS3FullAccessRole
- Copy ARN to use in the next pages
- Back to the CloudFormation console - click in AWS principal link
- CloudFormation
- Create Stack
- Upload a Template to Amazon S3

lambda-s3-resource-helloworld.yml
```yml
Description: >
  Simple custom resource demo
Parameters:

  InputMessage:
    Type: String
    Description: An input to the custom resource
    Default: Hello Function!!

  RoleForLambda:
    Description: ARN or the role you created
    Type: String

Resources:

  MyCustomResourceFunction:
    Type: AWS::Lambda::Function
    Properties:
      Code:
        ZipFile: |
          const response = require('cfn-response');
          const aws = require('aws-sdk');
          exports.handler = (event, context) => {
            const input = event.ResourceProperties.InputParameter;
            const responseData = { msg: 'hello world', msg2: `${input} --received from caller`};
            response.send(event, context, response.SUCCESS, responseData);
          };

      Handler: index.handler
      Timeout: 30
      Runtime: nodejs12.x
      Role: !Ref RoleForLambda

  MyCustomResourceCallout:
    Type: Custom::LambdaCallout
    Properties:
      ServiceToken: !GetAtt MyCustomResourceFunction.Arn
      InputParameter: !Ref InputMessage

Outputs:
  OutputFromFunction:
    Description: Output from the custom Function
    Value: !GetAtt MyCustomResourceCallout.msg

  ModifiedInputReturned:
    Description: Pipe out the input so we know we got it
    Value: !GetAtt MyCustomResourceCallout.msg2
```

- CloudFormation console
- upload lambda-s3-resource.yml file
- Next
- Set Role: arn:aws:iam::620157586684:role/LambdaS3FullAccessRole
- Name custom-resource-demo-lambda
- Next
- Next
- Create Stack
- Solve Conflict of Typo, like nodejs12.0 is not good, instead nodejs12.x
- CREATED COMPLETE, this is OK.

Success!! AWS Lambda.

- Now Delete this Stack it to use it with other resource.

```
| REMEMBER: CloudFormation -> Delete Stack, so you have no extra charge. |
```

lambda-custom-resource-final.yml
```yml
Description: >
  Simple custom resource demo
Parameters:

  BucketName:
    Type: String
    Description: The name of an S3 bucket

  RoleForLambda:
    Description: ARN or the role you created
    Type: String

Resources:

  MyCustomResourceFunction:
    Type: AWS::Lambda::Function
    Properties:
      Code:
        ZipFile: |
          const response = require('cfn-response');
          const aws = require('aws-sdk');
          exports.handler = (event, context) => {
            const responseText = 'Starting Function';
            const s3 = new aws.S3();
            const bucketName = event.ResourceProperties.BucketName;
            if( event.RequestType == 'Create'){
              s3.createBucket({Bucket: bucketName}, function(err, data){
              });
            } else if( event.RequestType == 'Delete' ){
              s3.deleteBucket({Bucket: bucketName}, function(err, data){
              });
            }
            const responseData = { msg: 'hello S3 Resource!!', responseText: responseText };
            response.send(event, context, response.SUCCESS, responseData);
          };
      Handler: index.handler
      Timeout: 30
      Runtime: nodejs12.x
      Role: !Ref RoleForLambda

  MyCustomResourceCallout:
    Type: Custom::LambdaCallout
    Properties:
      ServiceToken: !GetAtt MyCustomResourceFunction.Arn
      InputParameter: !Ref BucketName

Outputs:
  OutputFromFunction:
    Description: Output from the custom Function
    Value: !GetAtt MyCustomResourceCallout.msg

  ResponseText:
    Description: Output custom Function
    Value: !GetAtt MyCustomResourceCallout.responseText
```

```
| REMEMBER: CloudFormation -> Delete Stack, so you have no extra charge. |
```
