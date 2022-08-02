# 모든 AWS region의 AWS config를 Enable 하는 스크립트

~~~bash
#!/bin/bash

PROFILE="vcs-prod"
ARN="arn:aws:iam::123456789:role/aws-service-role/config.amazonaws.com/AWSServiceRoleForConfig"
S3BUCKET="config-bucket-123456789"
REGION=( eu-north-1 ap-south-1 eu-west-3 eu-west-2 eu-west-1 ap-northeast-3 ap-northeast-2 ap-northeast-1 sa-east-1 ca-central-1 ap-southeast-1 ap-southeast-2 eu-central-1 us-east-1 us-east-2 us-west-1 us-west-2)

## delivery 채널 파일 생성
cat > deliveryChannel.json << EOF
{
    "name": "default",
    "s3BucketName": "${S3BUCKET}",
    "configSnapshotDeliveryProperties": {
        "deliveryFrequency": "Twelve_Hours"
    }
}
EOF

## main

for i in ${REGION[*]}
do

echo "### Region : $i"

## aws config 레코더 생성
echo "### create aws config recoder ###"
aws --profile ${PROFILE} --region ${i} \
configservice put-configuration-recorder \
--configuration-recorder name=default,roleARN=${ARN} \
--recording-group allSupported=true,includeGlobalResourceTypes=true

## 전송방법 설정
echo "### set delivery channel ###"
aws --profile ${PROFILE} --region ${i} \
configservice put-delivery-channel \
--delivery-channel file://deliveryChannel.json

## 기록 시작
echo "### start recoder ###"
aws --profile ${PROFILE} --region ${i} \
configservice start-configuration-recorder --configuration-recorder-name default

done
~~~
