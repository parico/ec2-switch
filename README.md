# ec2-switch #
画面なしでEC2の停止・開始を実行するだけのGoスクリプトです。

## Requirement　##
* AWS IAM Role (access to EC2)
* ec2-switch.go
* EC2タグの追加
* ローカルにgo環境（クロスコンパイル用）


## How to use ec2-switch ##

```
# without compiling
go run ec2-switch.go -o start
go run ec2-switch.go -o stop
go run ec2-switch.go -o status

# compiled
./ec2-switch -o start
./ec2-switch -o stop
./ec2-switch -o status
```

## Installing ec2-switch ##
### IAMロールを作成 ###
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "StmtXXXXXXXXXXX",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Sid": "StmtXXXXXXXXXX",
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Condition": {
                "StringEquals": {
                    "ec2:ResourceTag/Env": "dev",
                    "ec2:ResourceTag/Switch": "true"
                }
            },
            "Resource": [
                "*"
            ]
        },
        {
            "Sid": "StmtXXXXXXXXXXX",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

### 対象のEC2に指定のタグを追加 ###
| Key  | Value |
| ------------- | ------------- |
| Env  | dev |
| Switch  | true |

Envタグをdev以外に変更したい場合は、ソースの下記部を変更すること。
```
 dev := contains(tags, "Env", "dev")

```

### クロスコンパイル ###
```
GOOS=linux GOARCH=amd64 go build ec2-switch.go
```

### cronとして反映 ###
```
# ec2-switch start
00 7 * * 1-5 /opt/ec2-switch/bin/ec2-switch -o start

# ec2-switch stop
50 23 * * 1-5 /opt/ec2-switch/bin/ec2-switch -o stop
```
