# Evaluation BRUNEL Grégory


# VPC
Virtual Private Cloud, il permet au utilisateurs de créer un environnement cloud isolé et privé dans lequel ils peuvent déployer leurs ressources informatiques.

## Création vpc

aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query Vpc.VpcId --output yaml

[vpc-0960d9a9a4e425bcc]

## nom vpc
Pour changer le nom du vpc.

aws ec2 create-tags --resources vpc-0960d9a9a4e425bcc --tags Key=Name,Value=vpceval

# SUBNET
Un subnet divise un grand réseau informatique en parties plus petites pour une gestion et une organisation plus efficaces. 

## Subnet public
Ici, le subnet sort vers internet.

aws ec2 create-subnet --vpc-id vpc-0960d9a9a4e425bcc --cidr-block 10.0.1.0/24 --availability-zone us-east-2a --query Subnet.SubnetId --output yaml

[subnet-0fedea0f8a7077af2]

## Subnet private
Ici, le subnet n'a pas accès à internet.

aws ec2 create-subnet --vpc-id vpc-0960d9a9a4e425bcc --cidr-block 10.0.2.0/24 --availability-zone us-east-2a --query Subnet.SubnetId --output yaml

[subnet-0756bcfa30b0cfbe3]

## nom subnet
Pour changer le nom du subnet.

aws ec2 create-tags --resources subnet-0756bcfa30b0cfbe3 --tags Key=Name,Value=privateeval
aws ec2 create-tags --resources subnet-0fedea0f8a7077af2 --tags Key=Name,Value=publiceval

# GATEWAY
Une gateway est essentiellement une porte d'entrée ou de sortie dans un réseau informatique. C'est comme une porte qui permet aux données d'entrer ou de sortir du réseau vers d'autres réseaux ou l'internet.

## création de gateway

aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output yaml

[igw-0c9a63b50b2f807d4]

## nom gateway 
Pour changer le nom de la gateway.

aws ec2 create-tags --resources igw-0c9a63b50b2f807d4 --tags Key=Name,Value=gatewayeval

## attacher vpc gateway
Pour lier le vpc à la gateway

aws ec2 attach-internet-gateway --vpc-id vpc-0960d9a9a4e425bcc --internet-gateway-id igw-0c9a63b50b2f807d4

# TABLE DE ROUTAGE
Une table de routage est comme une carte pour les données dans un réseau. Elle indique aux données où aller lorsqu'elles voyagent à travers le réseau. C'est comme si chaque route avait une série d'indications pour savoir comment atteindre sa destination.

## création de la table de routage public

aws ec2 create-route-table --vpc-id vpc-0960d9a9a4e425bcc --query RouteTable.RouteTableId --output text

[rtb-07db43d832c581f0e]

## nom table de routage
Pour changer le nom de la table de routage

aws ec2 create-tags --resources rtb-07db43d832c581f0e --tags Key=Name,Value=routetableeval

## création de la table de routage privé
Table de routage qui n'a pas accès à internet

aws ec2 create-route-table --vpc-id vpc-0960d9a9a4e425bcc --query RouteTable.RouteTableId --output text

[rtb-0047df8abdba9c895]

## nom table de routage privé
Pour changer le nom de la table de routage

aws ec2 create-tags --resources rtb-0047df8abdba9c895 --tags Key=Name,Value=routetableeval2
# ROUTE
Une route est comme un itinéraire sur une carte qui montre comment aller d'un endroit à un autre. C'est un guide qui aide les données à voyager d'un appareil à un autre à travers un réseau informatique, comme des panneaux de signalisation sur une autoroute qui indiquent la direction à suivre.


## route passerelle nat
Une route passerelle NAT guide le trafic réseau sortant d'un réseau privé vers Internet en utilisant une adresse IP publique partagée, appelée adresse IP NAT.

aws ec2 create-route --route-table-id rtb-0047df8abdba9c895 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-0ec04fa471836e656

## creation de route

aws ec2 create-route --route-table-id rtb-07db43d832c581f0e --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0c9a63b50b2f807d4

# associate route

## public

aws ec2 associate-route-table --route-table-id rtb-07db43d832c581f0e --subnet-id subnet-0fedea0f8a7077af2

## private

aws ec2 associate-route-table --route-table-id rtb-0047df8abdba9c895 --subnet-id subnet-0756bcfa30b0cfbe3


# groupe de sécurité
Un groupe de sécurité est comme un gardien numérique qui contrôle qui peut entrer et sortir d'un endroit dans un réseau informatique. C'est comme une liste de contrôle d'accès numérique qui permet seulement aux bonnes personnes d'accéder à certaines parties du réseau et bloque tout ce qui pourrait être dangereux ou non autorisé, tout en assurant la sécurité des données.

## création de groupe de sécurité

aws ec2 create-security-group --group-name securitygroupeval --description "Groupe de securite eval" --vpc-id vpc-0960d9a9a4e425bcc

[sg-01dca64d976bb7582]

## configuration du groupe de sécurité

aws ec2 authorize-security-group-ingress --group-id sg-01dca64d976bb7582 --protocol tcp --port 22 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-id sg-01dca64d976bb7582 --protocol tcp --port 5085 --cidr 0.0.0.0/0

## passerelle nat et ip élastique
Une adresse IP élastique est comme une étiquette unique pour un appareil sur Internet, mais qui peut être déplacée d'un appareil à un autre. C'est comme si vous aviez une adresse postale qui pouvait être collée sur différents bâtiments selon vos besoins. Cela permet à un service en ligne d'être accessible de manière fiable, même si l'appareil sous-jacent change ou est mis à jour.

aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text --region us-east-2 --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=ipelasticeval}]'

[eipalloc-0d1d7a0b44a567bcc]

aws ec2 create-nat-gateway --subnet-id subnet-0fedea0f8a7077af2 --allocation-id eipalloc-0d1d7a0b44a567bcc

# INSTANCES
Une instance est comme un ordinateur virtuel que vous pouvez louer dans le cloud. C'est comme si vous aviez une machine toute prête à utiliser sur Internet pour faire fonctionner des logiciels ou des services. Les instances sont flexibles et peuvent être adaptées à différents besoins, comme un couteau suisse numérique que vous pouvez configurer pour exécuter différents programmes ou tâches.

## Création d'instances

## public
Qui a accès à internet

aws ec2 run-instances --image-id ami-0ec3d9efceafb89e0 --count 1 --instance-type t2.micro --key-name sshkey --security-group-ids sg-01dca64d976bb7582 --subnet-id subnet-0fedea0f8a7077af2 --associate-public-ip-address --tag-specifications --region us-east-2 'ResourceType=instance,Tags=[{Key=DebianServer ,Value=Beta}]'

## private
Qui n'a accès à internet

aws ec2 run-instances --image-id ami-0ec3d9efceafb89e0 --count 1 --instance-type t2.micro --key-name sshkey --security-group-ids sg-01dca64d976bb7582 --subnet-id subnet-0756bcfa30b0cfbe3 --tag-specifications --region us-east-2 'ResourceType=instance,Tags=[{Key=DebianServer ,Value=Beta}]'

# ACL 
Une ACL (Liste de Contrôle d'Accès) est un peu comme une liste de contrôle pour un réseau informatique. C'est comme un garde à l'entrée d'une fête qui vérifie qui est autorisé à entrer et qui ne l'est pas. Une ACL contrôle le trafic réseau en spécifiant les règles qui permettent ou bloquent certaines communications entre les appareils du réseau. Cela aide à sécuriser le réseau en autorisant seulement les interactions approuvées et en bloquant celles qui pourraient être malveillantes ou non autorisées.

## création d'ACL

aws ec2 create-network-acl --vpc-id vpc-0960d9a9a4e425bcc

## ajout de règle entrante

aws ec2 create-network-acl-entry --network-acl-id acl-029120078f8447628 --rule-number 115 --ingress --protocol tcp --rule-action allow --cidr-block 0.0.0.0/0 --port-range From=22,To=22

aws ec2 create-network-acl-entry --network-acl-id acl-029120078f8447628 --rule-number 120 --ingress --protocol tcp --rule-action allow --cidr-block 0.0.0.0/0 --port-range From=5085,To=5085

aws ec2 create-network-acl-entry --network-acl-id acl-029120078f8447628 --rule-number 125 --ingress --protocol tcp --rule-action allow --cidr-block 0.0.0.0/0 --port-range From=1024,To=65535