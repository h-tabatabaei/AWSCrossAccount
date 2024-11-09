# Cross Account Resource Sharing in AWS
## account details:
-   **Account-A:** this account is the source account which want to access some resources in other account.`accoutn B`
-   **Account-B:** or Desitnation account which has some resource that Account-A wants to access.
## Account-B configuration:
1. login to console with IAM user in account-B
2. go to IAM and create new role:
   1. choose `AWS ACCOUNT` from Trusted entity type.
   2. down the page select `Another AWS Account` and enter the account ID of account-A and click next.
   3. choose the proper permission that account-A will be used to access resources in Account-B. in my example I choosed `AdministratorAccess`. Then click next.
   4. Set a descriptive name and click **Create Role**. I set `crossAccountRoleForAccountA`.
3. update the newly created role:
   1. filter the role `crossAccontRoleForAccountA` and open it.
   2. In **Trust Relationships** tab click on `edit trust policy`.
   3. update the principal as follow:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
        "Effect": "Allow",
        "Principal": {
            "AWS": "arn:aws:iam::<accountA ID>:user/<IAM user on account-A>"
        },
        "Action": "sts:AssumeRole",
        "Condition": {}
    }
  ]
}
```
4. note the arn of the role. you need this arn for next step.

## Account-A configuration:
1. login to console with IAM user in account-A
2. go to IAM console and create a new policy.
   1. open json editor and paste the json as follow and then click next:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": [
                "arn:aws:ia..<role arn of account-A>"
            ]
        }
    ]
}
```
   2. set a name for the policy and click **Create Policy**
3. Attach policy to the user which you want to have permission over account-B resources. 
Note: This user must be as same user as you set in `Account-B configuration` Step 3.3.
## Test the permission
1. Run `aws configure` and set access key and secret key of account-a IAM user.
2. edit the ~/.aws/config and add a profile as follow:
```
[profile prodaccess]
    role_arn = arn:aws:iam::<account-B ID>:role/crossAccontRoleForAccountA
    source_profile = default
```
Note: you need to add role arn which you define on account-B
3. now you can run a command. in the command below we are trying to add a record to a public hosted zone on Account-B from IAM user on Account-A
```
aws route53 change-resource-record-sets \
  --hosted-zone-id Z0.......VP --profile prodaccess \
  --change-batch '{
    "Comment": "Testing cross-account access",
    "Changes": [
      {
        "Action": "CREATE",
        "ResourceRecordSet": {
          "Name": "test.example.com",
          "Type": "A",
          "TTL": 300,
          "ResourceRecords": [
            {
              "Value": "192.0.2.44"
            }
          ]
        }
      }
    ]
  }'
```
