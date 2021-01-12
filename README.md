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

