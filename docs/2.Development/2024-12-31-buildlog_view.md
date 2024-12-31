# AWS 꿀팁

## 1. AWS Console에서 build log 보기 


~~~
#!/bin/bash


build_project="${BUILD-PROJECT-NAME}"

build_latest=`aws codebuild list-builds-for-project --project-name ${build_project} --output text |head -1 | awk -F[:] '{print $2}'`

echo $build_latest

aws logs get-log-events --log-group-name /aws/codebuild/${build_project} --log-stream-name logs/${build_latest} | jq .events[].message
~~~
