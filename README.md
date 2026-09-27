# AWS ECS Project
>In this project, we are leveraging Amazon ECS, AWS’s fully managed container orchestration service, to deploy a microservices‑based architecture. The setup includes two containers: a Node.js application that exposes RESTful APIs supporting GET, POST, and PATCH methods, and a MongoDB container serving as the backend database.

# Architecure

In this project we will utilized following AWS services.</br>
• ****VPC & Subnet** -** Provides a secure, isolated network environment to host containers with fine‑grained control over IP addressing and routing.</br>
• **Security Groups -** Acts as a virtual firewall to control inbound and outbound traffic, ensuring only authorized access to application and database containers.</br>
• **ECS (Elastic Container Service)  -** Orchestrates and manages containerized workloads, enabling seamless deployment and scaling of the Node.js API and MongoDB services.</br>
• **ALB (Application Load Balancer) -** Distributes incoming traffic across containers, ensuring high availability, fault tolerance, and efficient request handling.</br>
• **EFS (Elastic File System) -** Offers scalable, shared storage for containers, supporting persistent data needs and simplifying stateful application management.</br>
<img width="665" height="625" alt="image" src="https://github.com/user-attachments/assets/5093a1a5-488b-4bdb-b4f7-267c68cc4e56" />



# Implementation Steps
**Docker Image** </br>
• DockerFile is available use following command to create image. Note Docker Desktop should be running. </br>
• docker build -t aws_ecs_project:latest . </br>
This will create image in local docker. </br>
• Create public repository in remote desktop</br>
<img width="1520" height="360" alt="image" src="https://github.com/user-attachments/assets/9d7dd2b6-597e-4f3a-99ff-98f0726fec3c" />
• Tag image - docker tag aws_ecs_project:latest mohsiuddinjagirdar/nodejs-image:latest</br>
• Push the image to remote desktop mohsiuddinjagirdar/nodejs-image:latest</br>
<img width="1542" height="762" alt="image" src="https://github.com/user-attachments/assets/f509cf8f-8e37-4aa0-ac74-c67b6a933989" />


**VPC and Subnet Creation** </br>
In this project we will create custom VPC rather then working on default VPC.</br>
• Create VPC, keep CIDR reasonable like 10.0.0.0/22 - this blocks around 1024 IPs.
<img width="713" height="511" alt="image" src="https://github.com/user-attachments/assets/133cd007-318e-435e-afe3-b3521f4c4956" /></br>
• Edit VPC Setting and enable DNS hostnames</br>
<img width="1102" height="91" alt="image" src="https://github.com/user-attachments/assets/05af08b3-001f-46d8-8822-3af4d6617ee9" />
• Create 2 subnets with CIDR /24 this blocks 256 IPs per subnet. Use 10.0.0.0/24 & 10.0.1.0/24. Keep Availability zone different like 1a, 1b. You can create 4 subnets if required.</br>
<img width="1097" height="773" alt="image" src="https://github.com/user-attachments/assets/ab8fd8eb-e51c-47c6-9fc1-12267b535000" />

• Create Internet Gateway to allow connection to internet.</br>
<img width="955" height="317" alt="image" src="https://github.com/user-attachments/assets/6ff6198a-66ec-461d-a3c0-40275f05e35b" />
• Attach the Internet Gateway to the VPC created.</br>
• In route table for VPC add route to internet(0.0.0.0/0) from internet gateway.</br>
<img width="1112" height="335" alt="image" src="https://github.com/user-attachments/assets/ec1c02a5-5cbd-4b03-8092-c2d894090a98" />

**EFS Creation** </br>
We will use EFS as persistent data storage for mongodb container.</br>
• Provide EFS name ("mongo-db"), Provide the vpc and subnet created for project. Create a sepparate security group which allow inbound from VPC SG on TCP port 2049 NFS port.</br>
<img width="1098" height="601" alt="image" src="https://github.com/user-attachments/assets/709f286e-59df-4b9b-b55f-2c1b7cd97147" /></br>
<img width="1090" height="322" alt="image" src="https://github.com/user-attachments/assets/e84a7ebb-ae19-436c-a382-0725bcfc4aac" />

**ECS Creation** </br>

**TASK CREATION**
• First create a Task Definition which is equivalent of docker compose where we define all requirement for running a container. This task definition will contain 2 images, volume details and volume mount point. </br>
• Provide Task Definition name.</br>
• Launch Type- Farget</br>
• Operating system/Architecture - Linux</br>
• CPU - 1 vCPU, Memory - 2GB </br>
• Task execution role - Create a default role.</br>
<img width="1102" height="725" alt="image" src="https://github.com/user-attachments/assets/0c42f58e-e5fd-4059-83f0-0d7d12f2bdfd" />

Container – 1</br>
• name: mongo</br>
• image: mongo. This will fetch image from docker hub.</br>
• Container port: 27017</br>
• Add Env Variable - </br>
  MONGO_INITDB_ROOT_PASSWORD - password</br>
  MONGO_INITDB_ROOT_USERNAME - mongo</br>
<img width="1100" height="801" alt="image" src="https://github.com/user-attachments/assets/30bd45b7-978d-4a04-b140-550b9af2fce3" />

Container – 2</br>
• Name - web-api</br>
• Image URI - mohsiuddinjagirdar/nodejs-image (Use your public repo used for image promotion)</br>
• Container port - 3000</br>
• Add Env Variable - </br>
  MONGO_USER - mongo</br>
  MONGO_PASSWORD - password</br>
  MONGO_IP - localhost</br>
  MONGO_PORT - 27017</br>
<img width="1113" height="817" alt="image" src="https://github.com/user-attachments/assets/04997ff1-5cfe-4e70-ba91-97e2835104ca" />

Add Storage </br>
• Add Volume to EFS created.</br>
• Mount the volume to mongo container at path /data/db</br>
<img width="1092" height="808" alt="image" src="https://github.com/user-attachments/assets/8704ae7a-8de0-458a-8c73-90baffb7936d" />

**Cluster Creation and Service creation.**</br>
• Provide cluster name and create Fargate cluster.</br>
<img width="1115" height="605" alt="image" src="https://github.com/user-attachments/assets/3645ad2f-a5a0-4534-b352-874e9ddea7bc" />

• Inside cluster create service</br>
• Task definition family - Task created earlier.</br>
• Task definition revision</br>
• Service name - Provide service name </br>
• Deployment configuration - Replica</br>
• Desired tasks - 1 . Number on container we need.</br>
• Networking - Add proper VPC and subnets. Add default security group.</br>
• Create Load balancer using following settings, select web-api container and traffic need to be forwarded on that port. the Load balancer security group should allow traffic from Internet on port 80</br>
<img width="1090" height="817" alt="image" src="https://github.com/user-attachments/assets/9cf64dff-dd99-4a93-983b-a372429b0018" />
<img width="818" height="643" alt="image" src="https://github.com/user-attachments/assets/a66affeb-08c2-4dcd-9d3e-c65d9f907faa" />
<img width="828" height="733" alt="image" src="https://github.com/user-attachments/assets/54c0d44d-3130-4dc2-9a56-7e464611082e" />
<img width="1337" height="551" alt="image" src="https://github.com/user-attachments/assets/0daece3e-d91f-4382-9692-dc682ce13124" />
<img width="1323" height="607" alt="image" src="https://github.com/user-attachments/assets/592a1ca7-5133-48c5-bcb1-e7d7eecb02af" />
<img width="1320" height="381" alt="image" src="https://github.com/user-attachments/assets/8e02217f-2c65-4cad-b8d0-54a19a3ac358" />


# Validation

**ECS Verification**
<img width="1667" height="422" alt="image" src="https://github.com/user-attachments/assets/00619a4a-0fe4-401a-aa7a-2c96ad7056de" />

<img width="1675" height="428" alt="image" src="https://github.com/user-attachments/assets/f3a13d44-a04c-4ccd-ae52-f7f353256c95" />

<img width="1682" height="463" alt="image" src="https://github.com/user-attachments/assets/4f811794-b414-4fdf-af18-456d6579276d" />

<img width="1672" height="617" alt="image" src="https://github.com/user-attachments/assets/b7fc8a05-bf5b-4b3b-8be0-6f7366de8116" />

<img width="1687" height="268" alt="image" src="https://github.com/user-attachments/assets/dae4f531-4dc2-41ab-a73d-cd21ffc156a7" />


**Access Load balancer URL to view GET response**

<img width="1106" height="350" alt="image" src="https://github.com/user-attachments/assets/32185edd-67ba-4cee-9928-215244d89709" />

**Use Post method to add value.**
<img width="1070" height="681" alt="image" src="https://github.com/user-attachments/assets/5bc0d0fd-5151-497f-ac66-d20e009e5391" />

<img width="1050" height="766" alt="image" src="https://github.com/user-attachments/assets/2c5794b6-15cc-4993-b122-b2e476ded159" />

# CleanUp
• Delete ECS Service.</br>
• Delete ECS Cluster.</br>
• Delete ECS Task Definition.</br>
• Delete EFS.</br>
• Delete load balancer, target groups</br>
• Delete Internet Gateway.</br>
• Delete VPC and subnets.</br>
• Delete Security groups.</br>



































