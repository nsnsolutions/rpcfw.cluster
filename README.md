# Launch a new cluster.

The following describes how to launch a new ECS Cluster in amazon. If you are
adding nodes to an exsiting cluster, you can skip to [Create ec2 Instances]. It
is easer to _Launch more like this_ from EC2 and would be the prefered method.

## Create ECS Cluster

We will create a new cluster in Amazon ECS. This is a logical group of servers
that share resources to execute tasks.  You can skip this task if you already
have a cluster created.

- Go to [Amazon ECS](https://us-west-2.console.aws.amazon.com/ecs/)
- Verify your region is what you want.
- Click _Create Cluster_
- Type in a cluster name and save it. We will refer to this name as `CLUSTER`

## Get a CoreOS Cluster Token

The CoreOS Cluster token is actually a discovery token from _etcd.io_. This
token tells new servers how to connect to each other. You need to save this
token in case you want to add servers to this cluster. It is the only way to
join more servers so put it some place safe.

- Create a new cluster token. (click one of the links below)
  - [1 Server](http://discovery.etcd.io/new?size=1)
  - [3 Server](http://discovery.etcd.io/new?size=3)
  - [5 Server](http://discovery.etcd.io/new?size=5)
  - [7 Server](http://discovery.etcd.io/new?size=7)
- Save the token. We will refer to this token as `TOKEN`

## Create the Cloud-Init.yaml file

If you have a template engine installed, you can populate the tokens with that.
The ecs-rabbitmq-cloud-init.j2 template is a jinja2 template.

- Save a copy the ecs-rabbitmq-cloud-init.j2 template named `user-data.yaml`
- Repalce all instances of `{{ TOKEN }}` with `TOKEN`
- Repalce all instances of `{{ CLUSTER }}` with `CLUSTER`
- Save the changes

## Create ec2 Instances

If you are creating a ec2 instance to add to an exsiting cluster, it is best to
select _Create more like this_ from the ec2 console.

- Go to [Amazon Ec2](https://us-west-2.console.aws.amazon.com/ec2/)
- Verify your region is what you want.
- Click _Launch Instance_
- Click _Community AMIs_
- Search for ami-06af7f66
- Click _Select_ on the __CoreOS-Stable-*-hvm__ image.
- Select the server type. Example: t2.micro is good for development.
- Click _Next: Configure Instance Details_
- Fill out the form.
  - Number of instances: If deploying a new cluster, this should match the number of servers from [Get a CoreOS Cluster Token]. Else, whatever you need.
  - Verfiy _Network_, _IAM Role_, _Shutdown Behavior_.
- Exampand _Advanced Details_
  - Copy the contence of user-data.yaml from step [Create the Cloud-Init.yaml file]
  - Paste the file in the _User data_ field.
- Click _Next: Add Storage_
- Click _Next: Tag Instance_
- Provide a name. Example `ECS.Node (Development)` or `ECS.Node (Production)`
- Click _Next: Configure Security Group_
- Select the appropriate security group.
- Click _Review and Launch_
- Double check your configuration and click "Launch"

_Note: The Following ports to be accessable between nodes: 1883 2379 2380 4001
4369 7001 8883 61613 61614._

## Verify cluster

It might take about a minuit for the new hosts to become available.

- Go to [Amazon Ec2](https://us-west-2.console.aws.amazon.com/ec2/)
- Verify your new instances are starting up.
- Select connect on any node and copy the public DNS name.
- SSH into the server and type `etcdctl cluster-health`. You should see _Cluster is healthy_
- Go to [Amazon ECS](https://us-west-2.console.aws.amazon.com/ecs/)
- Make sure the number of servers reflects the changes you made.

You Did It!
