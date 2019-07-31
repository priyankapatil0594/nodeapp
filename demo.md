\## RabbitMQ

 

RabbitMQ is an open-source message broker that offers reliable, highly available, scalable, and portable messaging.

 

Message queues provide an asynchronous communication mechanism in which the sender and the receiver of a message do not contact each other. They also do not need to communicate with the message queue at the same time either. When a sender places a messages onto a queue, it is stored until the recipient receives them.

 

![](hello-world-example-routing.png)

 

The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue. Instead, the producer can only send messages to an exchange. An exchange is a very simple thing; one side receives messages from producers and the other side pushes them to queues. Then the consumer receives messages from the queue.

 

 

 

 

\#### **Install Node.js**

 

\```execute

sudo yum install nodejs 

\```

 

 

Start execution by login.

 

\```execute

oc login -u admin -p Passw0rd

\```

Let's create a new project.

 

\```execute



oc new-project nodejs-project

oc project nodejs-project

\```

create the nodejs application.

 

\```execute

oc new-app nodejs-app

\```

 

 

Import the application template into OpenShift.

 

\```execute

git clone https://github.com/jharting/openshift-rabbitmq-cluster.git

\```

 

\#Navigate to openshift-rabbitmq-cluster directory

 

\```execute

cd openshift-rabbitmq-cluster

\```

 

\```execute

oc create -f rabbitmq-cluster-template.yaml

\```

 

Output:

 

\> template.template.openshift.io/rabbitmq-cluster created

 

 

 

To process the template and create a new cluster in the current project with the default settings, run the below command (we create a RabbitMQ service from yaml file that we cloned before):

 

\```execute

oc process -f rabbitmq-cluster-template.yaml NAMESPACE="$(oc project --short)" RABBITMQ_PASS=password123 | oc create -f -

\```

 

Here 'RABBITMQ_PASS =password123' is password that is used while connecting producer and consumer to rabbitmq server with default username rabbitmq

 

\> serviceaccount/rabbitmq-discovery created

\> rolebinding.authorization.openshift.io/rabbitmq-discovery-view created

\> secret/rabbitmq-cluster-secret created

\> configmap/rabbitmq-cluster-config created

\> service/rabbitmq-cluster-balancer created

\> service/rabbitmq-cluster created

\> statefulset.apps/rabbitmq-cluster created

 

To see the pod details

 

\```execute

oc get pod

\```

 

Output:

 

\> NAME                           READY     STATUS    RESTARTS   AGE

\> rabbitmq-cluster-0           1/1       Running     0          11m

\> rabbitmq-cluster-1           1/1       Running     0          10m

 

 

 

Now we need to deploy producer and consumer application.The following images displays the flow of deploying producer and consumer apps

 

![](Workflow.png)

 

As shown above the entire process can be summarized into four steps

 

\1. Cloning Github repository

\2. Building Images from cloned directory(producer , consumer)

\3. Pushing built images to Openshift Internal registry(producer , consumer)

\4. pulling images to Openshift project from openshift internal registry(producer , consumer)

 

\### 1.Cloning Github repository

 

Now we are going to deploy the producer application.

 

Before that we need to download the producer and consumer code

 

\```execute

curl  -O -L https://raw.githubusercontent.com/snippet-java/ops/master/odo-demo/rabbitmq/rabbitmq.zip 

\```

 

Unzip the downloaded file

 

\```execute

unzip rabbitmq.zip

\```

 

Move to code directory

 

\```execute

cd rabbitmq

\```

 

It has both producer and consumer codes 

 

\```execute

ls

\```

 

\> consumer  producer

 

\### 2.Building Image from cloned directory(producer)

 

Now move to the producer directory to build using docker file

 

\```execute

cd producer

\```

 

It has Dockerfile and index.py file

 

\```execute

ls

\```

 

\> index.py  Dockerfile

 

Now message in index.py is "Hello World!'"

 

If you want to change the message open the index.py by **vi index.py** and edit the message in  channel.basic_publish(body="Your own message") and save the file (or) you can skip this step.

 

Now build the docker image of producer(prod is image name)

 

\```execute

docker build -t prod .

\```

 

\> Sending build context to Docker daemon 3.584 kB

\> Step 1/10 : FROM python:2

\>  ---> 3edf9092825f

\> Step 2/10 : RUN pip install rabbitmq

\>  ---> Running in 0fd4ef17e466

\> .....................

\> Removing intermediate container ae0f5f653903

\> Step 8/9 : EXPOSE 8080

\> ---> Running in af6b648d31cc

\> ---> b30dc6e8055c

\> Removing intermediate container af6b648d31cc

\> Step 9/9 : CMD python /index.py $ipa

\> ---> Running in b28b78af703e

\> ---> 52145abab894

\> Removing intermediate container b28b78af703e

\> Successfully built 52145abab894

 

\### 3.Pushing built image to Openshift Internal registry(producer)

 

Now push the image to Openshift internal registry (Later we can pull image from this registry)

 

For that we need to get the Name and token id of current user

 

\```execute

oc whoami

\```

 

\> admin

 

\```execute

token_id=$(oc whoami -t )

\```

 

\```execute

echo $token_id

\```

 

output:

 

\> 1CEv22yFV0_bQnXg4Y_PGaQP7eArTarJ5dDxucJ2pKw

 

Login to the internal registry (Internal registry name is 'docker-registry.default.svc:5000' )

 

\```execute

docker login -u admin -p $token_id docker-registry.default.svc:5000

\```

 

\> Login Succeeded

 

Now tag the created docker image to this registry with some tag name such as 'producerRab'

 

\```execute

docker tag prod docker-registry.default.svc:5000/default/python:producerRab

\```

 

Here /default/python is path of internal registry

 

Now push the docker image to internal registry 

 

\```execute

docker push docker-registry.default.svc:5000/default/python:producerRab

\```

 

 

 

\> The push refers to a repository [docker-registry.default.svc:5000/default/python]

\> f693dbd61660: Pushed

\> 1401a2e969f9: Pushed

\> c0d9990b3f44: Pushed

\> 85e5374e8dd3: Pushed

\> 5c46a470c0e3: Pushed

\> d3e157eb7218: Pushed

\> c9007a02e2d4: Pushed

\> 6e5de71de0af: Pushed

\> 8c487c756d71: Pushed

\> 05c027e771c8: Pushed

\> e9313b51f46d: Pushed

\> 46601dcd4114: Pushed

\> 31b0e148310d: Pushed

\> producer: digest: sha256:d8df6d2288344588b0ccccbf8b73292d0b9b35c11132f22d6a1c3d905cc2280e size: 3270

 

 

 

\### 4.Pulling image to Openshift project from openshift internal registry(producer)

 

Now pull the image(from the same registry we tagged above)  into our project 

 

For that we need to add pull policy to our project(rabbitmq-project)

 

\```execute

oc policy add-role-to-user system:image-puller system:serviceaccount:rabbitmq-project:default -n default

\```

 

\> role "system:image-puller" added: "system:serviceaccount: rabbitmq-project:default"

\> 

 

Now create the producer application

 

\```execute

oc new-app docker-registry.default.svc:5000/default/python:producerRab -e service='rabbitmq-cluster rabbitmq password123' --name=producer

\```

 

Note: Here service=rabbitmq-cluster refers to service name of  rabbitmq server  ,"rabbitmq" and "password123" are username and password for connecting to rabbbitmq server , producerRab is internal repository name and producer is the name producer image

 

 

 

\> --> Found Docker image 846558f (22 minutes old) from docker-registry.default.svc:5000 for "docker-registry.default.svc:5000/default/python:producerRab"

\> 

\> ```

\>* This image will be deployed in deployment config "producer"

\> * Port 8081/tcp will be load balanced by service "producer"

\>   * Other containers can access this service through the hostname "producer"

\> * WARNING: Image "docker-registry.default.svc:5000/default/python:producer" runs as the 'root' user which may not be permitted by your cluster administrator

\> ```

\> 

\> --> Creating resources ...

\>deploymentconfig.apps.openshift.io "producer" created

\> service "producer" created

\>  --> Success

\>  Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:

\> 'oc expose svc/producer'

\>  Run 'oc status' to view your app.

 

 

 

Now expose the app to create url for producer

 

\```execute

oc expose svc/producer

\```

 

\> route.route.openshift.io/producer exposed

 

 

 

To get the URL 

 

\```execute

oc get routes

\```

 

\> NAME       HOST/PORT                                              PATH      SERVICES   PORT       TERMINATION   

\> producer   producer-rabbitmq-project.apps.openshift185.bluecloudcourse.com             producer   8080-tcp                 

 

 

 

The URL **producer-rabbitmq-project.apps.openshift185.bluecloudcourse.com**  is to show the message was sent or not .

 

First Open web console and allow insecure traffic in routes  section to access the webpage.

 

You can do this by 

 

Routes > producer > Edit(top right corner) you need to click security checkbox and in Insecure Traffic click allow

 

Now you can open webpage:

 

\```execute

curl producer-rabbitmq-project.apps.openshift185.bluecloudcourse.com   

\```

 

\> [x] Sent 'Hello World!'

 

Now producer has sent a message. But Internally this message will be sent to rabbitmq's exchange , that will take care of sending message to consumer. Follow the below steps of creating consumer that will display the message sent by producer.

 

 

 

\#We follow same process for consumer as we did for producer in above steps

 

\### 2.Building Image from cloned directory(consumer)

 

Move to the consumer directory

 

\```execute

 cd .. && cd consumer

\```

 

Now build the docker image of consumer

 

\```execute

docker build -t crod .

\```

 

\> Sending build context to Docker daemon 3.584 kB

\> Step 1/9 : FROM python:2

\> ---> 3edf9092825f

\> Step 2/9 : RUN pip install rabbitmq

\> ---> Using cache

\> ---> 38a79490ef0f

\> Step 3/9 : RUN pip install Flask

\> ---> Using cache

\> 

\> . . . . . .

\> 

\> Removing intermediate container 189945a794b8

\> Step 8/9 : EXPOSE 8080

\> ---> Running in ae179124bb9e

\> ---> 60cdd20ea205

\> Removing intermediate container ae179124bb9e

\> Step 9/9 : CMD python /index.py $ipa

\> ---> Running in 9f2491401d0f

 

\### 3.Pushing built image to Openshift Internal registry(consumer)

 

Now tag the created docker image to internal registry with some tag name such as 'consumerRab'

 

\```execute

docker tag crod docker-registry.default.svc:5000/default/python:consumerRab

\```

 

Now push the docker image to internal registry 

 

\```execute

docker push docker-registry.default.svc:5000/default/python:consumerRab

\```

 

 

 

\> The push refers to a repository [docker-registry.default.svc:5000/default/python]

\> 26973365d439: Pushed

\> 1401a2e969f9: Layer already exists

\> c0d9990b3f44: Layer already exists

\> 85e5374e8dd3: Layer already exists

\> 5c46a470c0e3: Layer already exists

\> d3e157eb7218: Layer already exists

\> c9007a02e2d4: Layer already exists

\> 6e5de71de0af: Layer already exists

\> 8c487c756d71: Layer already exists

\> 05c027e771c8: Layer already exists

\> e9313b51f46d: Layer already exists

\> 46601dcd4114: Layer already exists

\> 31b0e148310d: Layer already exists

\> consumer: digest: sha256:2d78c044135984f2d3249aa347fce53c8f85e004c228b0cfd9f0843a1bb83265 size: 3059

 

\### 4.Pulling Image to Openshift project from openshift internal registry(consumer)

 

Now create the consumer application

 

\```execute

oc new-app docker-registry.default.svc:5000/default/python:consumerRab -e service='rabbitmq-cluster rabbitmq password123' --name=consumer

\```

 

Note: Here service=rabbitmq-cluster refers to service name of  rabbitmq server  ,"rabbitmq" and "password123" are username and password for connecting to rabbbitmq server , consumerRab is internal repository name and consumer is the name of consumer image

 

 

 

\> --> Found Docker image 0ef25c3 (9 minutes old) from docker-registry.default.svc:5000 for "docker-registry.default.svc:5000/default/python:consumerRab"

\> 

\> ```

\> * This image will be deployed in deployment config "consumer"

\> * Port 8080/tcp will be load balanced by service "consumer"

\>   * Other containers can access this service through the hostname "consumer"

\> * WARNING: Image "docker-registry.default.svc:5000/default/python:consumer" runs as the 'root' user which may not be permitted by your cluster administrator

\> ```

\> 

\> --> Creating resources ...

\>  deploymentconfig.apps.openshift.io "consumer" created

\>  service "consumer" created

\> --> Success

\>  Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:

\>   'oc expose svc/consumer'

\>  Run 'oc status' to view your app.

 

 

 

To expose the application

 

\```execute

oc expose svc/consumer

\```

 

\> route.route.openshift.io/consumer exposed

 

 

 

To get the URL

 

\```execute

oc get routes

\```

 

\> NAME       HOST/PORT                                              PATH      SERVICES   PORT       TERMINATION   WILDCARD

\> consumer   consumer-rabbitmq-project.apps.openshift185.bluecloudcourse.com             consumer   8080-tcp                 None

\> producer   producer-rabbitmq-project.apps.openshift185.bluecloudcourse.com             producer   8080-tcp   edge/Allow    None

 

 

 

Using this  consumer URL **consumer-rabbitmq-project.apps.openshift185.bluecloudcourse.com** we can see  received messages.

 

Open web console and allow insecure traffic in routes  section  to access the webpage.

 

 

 

To open webpage:

 

\```execute

curl consumer-rabbitmq-project.apps.openshift185.bluecloudcourse.com

\```

 

\> Hello World!

 

 

 

\#### Delete Rabbitmq**

 

Let's clean up the application and project:

 

\```execute

 oc delete all -l app=rabbitmq-cluster

\```

 

Sample output:

 

\> pod "rabbitmq-cluster-0" deleted

\> pod "rabbitmq-cluster-1" deleted

\> service "rabbitmq-cluster" deleted

\> service "rabbitmq-cluster-balancer" deleted

\> statefulset.apps "rabbitmq-cluster" deleted

 

\```execute

 oc delete project rabbitmq-project

\```

 

Sample output:

 

\> project.project.openshift.io "rabbitmq-project" deleted

 

\### **Summary**

 

We learnt about the creation of Rabbitmq server on Openshift,creation of Producer and Consumer .We understood  how  Producer and Consumer pods are configured to the RabbitMQ server to exchange messages between them.

 

