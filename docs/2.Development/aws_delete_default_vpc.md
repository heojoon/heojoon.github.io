# aws 기본 vpc를 삭제하는 python script 


~~~python
from multiprocessing import Pool
import boto3
import time

###########################################
# Delete default vpc 
###########################################

## aws profile 
awsProfile = "test-profile"


def remove_vpc(region):
    session = boto3.Session(profile_name=awsProfile)
    ec2 = session.resource('ec2',use_ssl=False,region_name=region)
    client = session.client('ec2',use_ssl=False,region_name=region)
    
        
    vpcs = ec2.vpcs.filter(Filters=[{'Name':'isDefault','Values':['true']}])
    
    print ("Region default vpc deleting...  :",region)
    for vpc in vpcs:
        
        # 0. vpc 확인
        print("[debug] default vpc : ", vpc.id)

        # 1. 인터넷 게이트웨이 제거
        igws = ec2.internet_gateways.filter(Filters=[{'Name':'attachment.vpc-id','Values':[vpc.id]}])
       
        for igw in igws:
            print ("internet gateway id:",igw)
            ec2.InternetGateway(igw.id).detach_from_vpc(VpcId=vpc.id)
            time.sleep(3)
            ec2.InternetGateway(igw.id).delete()
        
        # 2. 서브넷 제거
        subnets = ec2.subnets.filter(Filters=[{'Name':'vpc-id','Values':[vpc.id]}])
        
        for subnet in subnets:
            if subnet.id != "":
                ec2.Subnet(subnet.id).delete()
                print("[debug] subnet :" , subnet.id , "deleted")

        # 3. 보안그룹 제거 (default 외)
        security_groups = ec2.security_groups.filter(Filters=[{'Name':'vpc-id','Values':[vpc.id]}])
        
        for security_group in security_groups:
            if security_group.group_name != "default":
                ec2.SecurityGroup(security_group.id).delete()
                print("[debug] security group :" , security_group.id , "deleted")

        # 3. 라우팅테이블 제거
        # route_tables = client.describe_route_tables(Filters=[{'Name':'vpc-id','Values':[vpc.id]}])      
        # route_id = route_tables['RouteTables'][0]['RouteTableId']
        # print("[debug] route table id :",route_id)                
        # ec2.RouteTable(route_id).delete()
        
        # 4. VPC 삭제
        ec2.Vpc(vpc.id).delete()
        print(region, vpc.id, 'deleted')
            
        
if __name__ == '__main__':

    print("===================================")
    print("Delete aws default vpc program v1.0")
    print("===================================")
    regions = list(map(lambda x: x.get('RegionName'), boto3.Session(profile_name=awsProfile).resource('ec2',use_ssl=False).meta.client.describe_regions().get('Regions')))

    #print("region info : ",regions)
    
    # with Pool(4) as p:
    #     p.map(remove_vpc, regions)

    for i in regions:
        print("working.... :",i)
        remove_vpc(i)

~~~
