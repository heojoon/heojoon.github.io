- aws 기본 vpc를 삭제하는 python script 


~~~python
from multiprocessing import Pool
import boto3

###########################################
# Delete default vpc 
###########################################

## aws profile 
awsProfile = "test-profile"


def remove_vpc(region):
    session = boto3.Session(profile_name=awsProfile)
    ec2 = session.resource('ec2',use_ssl=False,region_name=region)
        
    vpcs = ec2.vpcs.filter(Filters=[{'Name':'isDefault','Values':['true']}])
    
    for vpc in vpcs:
        
        igws = ec2.internet_gateways.filter(Filters=[{'Name':'attachment.vpc-id','Values':[vpc.id]}])
        
        for igw in igws:
            ec2.InternetGateway(igw.id).detach_from_vpc(VpcId=vpc.id)
            ec2.InternetGateway(igw.id).delete()
            
        subnets = ec2.subnets.filter(Filters=[{'Name':'vpc-id','Values':[vpc.id]}])
        
        for subnet in subnets:
            ec2.Subnet(subnet.id).delete()
            
        ec2.Vpc(vpc.id).delete()
        
        print(region, vpc.id, 'deleted')
            
        
if __name__ == '__main__':

    regions = list(map(lambda x: x.get('RegionName'), boto3.Session(profile_name=awsProfile).resource('ec2',use_ssl=False).meta.client.describe_regions().get('Regions')))

    print("region info : ",regions)
    
    with Pool(4) as p:
        p.map(remove_vpc, regions)
~~~python
