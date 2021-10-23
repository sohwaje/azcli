#!/bin/bash

RgName="MyResourceGroup"
Location="koreacentral"
vm_name="my-vm"
vnet_name="my-vnet"
subnet_name="my-subnet"

# 리소스 그룹 생성
# az group create \
#   --name $RgName \
#   --location $Location

# 가상네트워크와 서브넷 생성
# az network vnet create \
#   --resource-group $RgName \
#   --name MyVnet \
#   --address-prefix 10.0.0.0/16 \
#   --location $Location \
#   --subnet-name MySubnet \
#   --subnet-prefix 10.0.0.0/24


# 가용성 집합 생성
az vm availability-set create \
  --resource-group $RgName \
  --location $Location \
  --name $vm_name-avs \
  --platform-fault-domain-count 2 \
  --platform-update-domain-count 2

# 로드밸런서에서 사용할 퍼블릭 IP 생성
az network public-ip create \
  --resource-group $RgName \
  --name $vm_name-lbip \
  --allocation-method Dynamic

# 로드밸런서 생성
az network lb create \
  --resource-group $RgName \
  --location $Location \
  --name $vm_name-lb \
  --frontend-ip-name FrontEnd \
  --backend-pool-name BackEnd \
  --public-ip-address $vm_name-lbip


# 프루브 생성(80, 443)
for i in 80 443; do
az network lb probe create \
  --resource-group $RgName \
  --lb-name $vm_name-lb \
  --name $i-Probe \
  --protocol http \
  --port $i --path /
done


# 로드밸런싱 규칙 생성(80, 443)
for i in 80 443; do
az network lb rule create \
  --resource-group $RgName \
  --lb-name $vm_name-lb \
  --name lbrbalancingrule-$i \
  --protocol Tcp \
  --probe-name $i-Probe \
  --frontend-port $i \
  --backend-port $i \
  --frontend-ip-name FrontEnd \
  --backend-pool-name BackEnd
done

# ############## VM 생성 ###############

# 네트워크 인터페이스 생성
for i in `seq 1 2`; do
az network nic create \
  --resource-group $RgName \
  --vnet-name $vnet_name \
  --subnet $subnet_name \
  --name $vm_name-$i-nic
done

# 로드밸런스에 연결할 IP 구성 생성
for i in `seq 1 2`; do
az network nic ip-config address-pool add \
  --address-pool BackEnd \
  --ip-config-name ipconfig1 \
  --lb-name $vm_name-lb \
  --nic-name $vm_name-$i-nic \
  --resource-group $RgName
done

# 가상머신 생성.
for i in `seq 1 2`; do
az vm create \
  --resource-group $RgName \
  --name $vm_name-$i \
  --nics $vm_name-$i-nic \
  --image UbuntuLTS \
  --availability-set $vm_name-avs \
  --admin-username azureadmin \
  --generate-ssh-keys
done
