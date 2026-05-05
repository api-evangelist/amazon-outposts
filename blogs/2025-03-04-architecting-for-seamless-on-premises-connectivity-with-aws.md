---
title: "Architecting for seamless on-premises connectivity with AWS Outposts servers"
url: "https://aws.amazon.com/blogs/compute/architecting-for-seamless-on-premises-connectivity-with-aws-outposts-servers/"
date: "Tue, 04 Mar 2025 21:45:47 +0000"
author: "aostan"
feed_url: "https://aws.amazon.com/blogs/compute/tag/aws-outposts/feed/"
---
<p><em>This post is written by Mark Nguyen, Principal Solutions Architect, AWS and Ryan Fillis, Solutions Architect, AWS.</em></p> 
<p><a href="https://aws.amazon.com/outposts/" rel="noopener" target="_blank">AWS Outposts</a> brings native AWS services, infrastructure, and operating models to virtually any data center, co-location space, or on-premises facility. Deploying <a href="https://aws.amazon.com/outposts/servers/" rel="noopener" target="_blank">Outposts servers</a> in your environment necessitates additional considerations regarding local network connectivity and <a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud (Amazon EC2)</a> instance networking. This post demonstrates the scalability of Outposts servers through automation and the deployment of Amazon EC2 network interfaces. This reduces the number of manual steps required to configure an Outposts server.</p> 
<p>This post details physically connecting your servers to your Local Area Network (LAN) and the networking options available for EC2 instances running on Outposts. We cover the physical cabling options, virtual networking components such as VPCs and subnets, and walkthrough an example setup for an EC2 instance with a user-data script to route traffic locally over your on-premises network.</p> 
<p>This post assumes that you have some familiarity with Outposts servers. If you would like a general refresher, observe <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/what-is-outposts.html" rel="noopener" target="_blank">What is AWS Outposts</a>. For more information about how to provision your Outposts server, see <a href="https://docs.aws.amazon.com/outposts/latest/install-server/install-server.html#install-network" rel="noopener" target="_blank">Installing an AWS Outposts server</a>.</p> 
<h2>Basic Amazon EC2 networking using a single interface</h2> 
<p>When launching an EC2 instance on an Outposts server a single interface is created for network connectivity. This default setting, depicted in the following diagram, is the most direct method for your instance to communicate externally.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2025/02/20/Figure-1-Simple-network-connectivity-on-an-Outposts-server.png"><img alt="Figure 1 Simple network connectivity on an Outposts server" class="size-full wp-image-23300 aligncenter" height="618" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2025/02/20/Figure-1-Simple-network-connectivity-on-an-Outposts-server.png" width="1144" /></a></p> 
<p style="text-align: center;"><em>Figure 1: Simple network connectivity on an Outposts server</em></p> 
<p>When deploying an EC2 instance to an Outposts server, there are certain differences in using the default <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html" rel="noopener" target="_blank">Elastic Network Interface (ENI)</a> as compared to deploying in an <a href="https://aws.amazon.com/about-aws/global-infrastructure/regions_az/" rel="noopener" target="_blank">AWS Region</a>. Understanding these differences is critical before modifying the network configuration, which you do in the next step.</p> 
<p>ENI differentiators between Outposts servers and the Region:</p> 
<ul> 
 <li><strong>Primary interface</strong>: The primary interface is an ENI. This ENI is associated to a subnet within a VPC. This VPC is extended from the Region to the Outposts server.</li> 
 <li><strong>IP address configuration</strong>: The primary network interface within the guest operating system (OS) of the EC2 instance must be configured to obtain an IP address through DHCP. The assigned IP address is from the IP address range of the VPC subnet associated with the Outposts server.</li> 
 <li><strong>Security group</strong>: A security group is associated with the ENI. This security group falls within the VPC that is extended from the Region. The user must apply appropriate access control rules to permit access to the EC2 instance. You may reuse security groups that already exist within the VPC.</li> 
 <li><strong>Outbound traffic</strong>: By default, an EC2 instance uses its ENI to direct outbound traffic toward the VPC subnet. Traffic flows according to the routing table associated with the Outposts server’s VPC subnet.</li> 
 <li><strong>Inbound traffic</strong>: If you’re only using an ENI, then traffic destined to EC2 instances on Outposts servers must traverse through the service link. In the preceding diagram, the user communicates with the EC2 instance over the internet. Traffic from the internet reaches the Region through the Internet Gateway of the VPC. Then, the VPC forwards the traffic to the appropriate subnet of the Outposts server (through the service link) and reaches the EC2 instance. The user must configure the necessary VPC components (Internet Gateway and associated routing table entries) for internet connectivity.</li> 
 <li><strong>Local network connectivity</strong>: There is no local network connectivity using the ENI. For local network connectivity, see the next section where we discuss the Outposts server <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/local-network-interface.html" rel="noopener" target="_blank">Local Network Interface (LNI)</a>.</li> 
</ul> 
<h2>Local network connectivity for EC2 instances</h2> 
<p>Outposts servers allow you to communicate through the <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/local-network-interface.html" rel="noopener" target="_blank">Local Network Interface (LNI)</a> in addition to the ENI. The LNI is a logical networking component that connects the Amazon EC2 instances in your Outposts subnet to your on-premises network.</p> 
<p>The Outposts server EC2 instance local communications characteristics:</p> 
<ul> 
 <li>Local network traffic needs the use of an LNI.</li> 
 <li>The subnets on Outposts servers must be enabled for LNIs. This is done by entering the following command:</li> 
</ul> 
<p><code>aws ec2 modify-subnet-attribute \</code></p> 
<p><code>--subnet-id <em>subnet-1a2b3c4d</em> \</code></p> 
<p><code>--enable-lni-at-device-index <em>1</em></code></p> 
<ul> 
 <li>IP address assignment for the LNI can be DHCP or static.</li> 
 <li>You can’t apply VPC security groups to the LNI. To control traffic on the LNI, you can use an OS based firewall, external on-premises firewall, or other security devices.</li> 
 <li><a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/outposts-cloudwatch-metrics.html" rel="noopener" target="_blank">Amazon CloudWatch</a> metrics are produced for each LNI.</li> 
 <li>Outposts servers don’t tag VLAN traffic. If VLAN tags are needed, then the network interface settings inside the guest OS must apply the VLAN tags. Multiple VLAN interfaces can exist within the same LNI (in this case you would be using the LNI as a VLAN trunk).</li> 
 <li>Local traffic bandwidth performance depends on the instance type. <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/local-network-interface.html" rel="noopener" target="_blank">The larger the instance type, the higher performance the throughput of the LNI</a>. The maximum throughput is 10 Gbps.</li> 
 <li>EC2 instances that communicate locally always have at least two interfaces: one ENI and one or more LNIs. Therefore, the instance OS’s routing table must be configured based on the desired traffic behavior.</li> 
</ul> 
<h2>Example configuration: Local traffic for EC2 instance on Outposts server</h2> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2025/02/20/Figure-2-Example-scenario-topology.png"><img alt="Figure 2 Example scenario topology" class="size-full wp-image-23299 aligncenter" height="814" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2025/02/20/Figure-2-Example-scenario-topology.png" width="1430" /></a></p> 
<p style="text-align: center;"><em>Figure 2: Example scenario topology</em></p> 
<p>In the example scenario, we want to launch an <a href="https://aws.amazon.com/linux/amazon-linux-2023/" rel="noopener" target="_blank">Amazon Linux 2023</a> instance and route all default traffic through the local network. Eth0 is the primary interface (ENI) and is used for traffic towards the Region. Eth1 is the LNI and is used for all other traffic. A <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html" rel="noopener" target="_blank">user-data script</a> is used to make the necessary routing changes at launch.</p> 
<p>Here is a sample user-data script. These commands run as root so there is no need to prepend each command with sudo.</p> 
<p>User data script (<code><em>my_userdata.txt</em></code>):</p> 
<pre><code class="lang-bash">#!/bin/bash 
route add -net 172.31.0.0/16 gw 172.31.239.1 
route del default gw 172.31.239.1 
cp -RL /run/systemd/network/* /etc/systemd/network/ 
echo -e '\n[Route]\nDestination=172.31.0.0/16\nGateway=172.31.239.1\nGatewayOnLink=yes' &gt;&gt; /etc/systemd/network/70-ens5.network 
sed -i -e 's/UseGateway=true/UseGateway=false/g' /etc/systemd/network/70- ens5.network.d/eni.conf</code></pre> 
<p>We can break down this script to observe the intent of each command:</p> 
<pre><code class="lang-bash">route add -net 172.31.0.0/16 gw 172.31.239.1 
route del default gw 172.31.239.1</code></pre> 
<p>When an instance is launched on Outposts server, the instance automatically has a default route that points toward the VPC through the ENI. In the example scenario, the desired configuration is to have all default traffic go through the LNI toward our on-premises LAN, not through the ENI. To accomplish this routing behavior for the ENI, we have to add a route toward the VPC and remove the default route. The first line adds a route through the VPC (172.31.0.0/16), using 172.31.239.1 as the gateway. The second line removes the default route that uses 172.31.239.1 (via the ENI) as the gateway.</p> 
<p>Traffic not destined for the VPC routes through the LNI. This includes all local traffic and internet-bound traffic. The local network’s <a href="https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html#DHCPOptionSets" rel="noopener" target="_blank">DHCP</a> server provides a default-gateway in its DHCP lease. Therefore, there is already a default route assigned to the LNI. This steers any traffic without a more specific route, including internet traffic, toward the LNI.</p> 
<p>Next, the user-data script makes the network settings persistent after reboot. The procedure varies depending on your OS. In the case of Amazon Linux 2023, it uses systemd-networkd.</p> 
<p><code>cp -RL /run/systemd/network/* /etc/systemd/network/</code></p> 
<p>This command copies the configuration files from the <code>/run/systemd/network/</code> folder to <code>/etc/systemd/network/</code>. The configuration files in the <code>/etc/systemd/network/</code> folder override the default settings and load during boot. The next is step is to modify the newly copied network configuration files.</p> 
<p><code>echo -e '\n[Route] \nDestination=172.31.0.0/16 \nGateway=172.31.239.1 \nGatewayOnLink=yes' &gt;&gt; /etc/systemd/network/70-ens5.network</code></p> 
<p>In this case the ENI is ens5. This line appends the static route section to the 70-ens5.network configuration file. This makes the static route added earlier in the script (route add -net 172.31.0.0/16 gw 172.31.239.1) persistent across reboots.</p> 
<p><code>sed -i -e 's/UseGateway=true/UseGateway=false/g' /etc/systemd/network/70- ens5.network.d/eni.conf</code></p> 
<p>Next, the user-script edits the configuration file, <code>eni.conf</code>, such that the default route isn’t used for the ENI at bootup. This is accomplished using sed to search and replace true with false for the <code>UseGateway</code> parameter.</p> 
<h2>Launching an instance with ENI and LNI</h2> 
<p>Now that the user-data script has been created, use the <a href="https://aws.amazon.com/cli/">AWS Command Line Interface (AWS CLI)</a> to launch an EC2 instance:</p> 
<pre><code class="lang-bash">aws ec2 run-instances \
--image-id ami-051f8a213df8bc089 \
--count 1 \
--instance-type c6id.xlarge \
--key-name my_key \
--user-data file://my_userdata.txt \
--network-interfaces '[ \
  { "DeviceIndex":0, "SubnetId":"subnet-0ca6abe6b34adfcce", "Groups": ["sg-0a9f8c2200c0a56f1"] }, \
  { "DeviceIndex":1, "SubnetId":"subnet-0ca6abe6b34adfcce", "Groups": ["sg-0a9f8c2200c0a56f1"] }]' \
--tag-specifications '[{ "ResourceType":"instance","Tags":[ \
  { "Key":"Name", "Value":"server1" } ] }]'</code></pre> 
<p>We can break down the parameters used in the preceding command:</p> 
<p><code>--image-id <em>ami-051f8a213df8bc089</em> \</code></p> 
<p>This specifies the <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html" rel="noopener" target="_blank">Amazon Machine Image (AMI)</a> ID. ami-051f8a213df8bc089 is the AMI ID for Amazon Linux 2023 in us-east-1.</p> 
<p><code>--count <em>1</em> \</code></p> 
<p>This specifies how many EC2 instances to launch. You can launch multiple at the same time.</p> 
<p><code>--instance-type c6id.xlarge</code></p> 
<p>This specifies the instance type. By default, Outposts 2U servers are slotted with the c6id.8xlarge instance type and Outposts 1U servers are slotted with the c6gd.8xlarge instance type. You can adjust the slotting assignment during the ordering process or you can change the slotting assignment later by using the <a href="https://aws.amazon.com/about-aws/whats-new/2024/11/self-service-capacity-management-aws-outposts/" rel="noopener" target="_blank">Self-service Capacity Management feature for AWS Outposts</a>.</p> 
<p><code>--key-name <em>my_key</em></code></p> 
<p>This specifies the public RSA key that is added to your EC2 instance. This key must already be defined in the same Region of your AWS account.</p> 
<p><code>--user-data file://<em>my_userdata.txt</em></code></p> 
<p>This specifies the filename that contains your user-data script (that was created previously).</p> 
<p><code>{ "DeviceIndex":0, "SubnetId":"<em>subnet-0ca6abe6b34adfcce</em>", "Groups": ["<em>sg-0a9f8c2200c0a56f1</em>"] }, \</code></p> 
<p><code>{ "DeviceIndex":1, "SubnetId":"<em>subnet-0ca6abe6b34adfcce</em>", "Groups": ["<em>sg-0a9f8c2200c0a56f1</em>"] }]' \</code></p> 
<p>This specifies the network interface configuration. By default, a single network interface, the ENI, is created. This example calls for a second interface for the LNI. DeviceIndex:0 is for the ENI and doesn’t change. DeviceIndex:1 is for the LNI, which we defined when we <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/local-network-interface.html" rel="noopener" target="_blank">enabled LNI for the subnet</a> (<code>--enable-lni-at-device-index 1</code>). The SubnetId refers to the subnet that was created on the Outposts server. If you want to deploy to a different Outposts server, then change the SubnetId. Groups refer to the security group that you would like assigned to the ENI. Security groups aren’t supported for the LNI, thus the security group specified for DeviceIndex:1 is only to comply with the command syntax check. A security group will not be applied to the LNI.</p> 
<p><code>--tag-specifications '[{ "ResourceType":"instance","Tags":[ \</code></p> 
<p><code>{ "Key":"Name", "Value":"server1" } ] }]'</code></p> 
<p>This assigns a name to the EC2 instance, which in this case is server1.</p> 
<h2>Conclusion</h2> 
<p>AWS Outposts servers allow you to run native AWS services on-premises by providing local compute. This supports workloads with low latency and data residency requirements through on-premises processing.</p> 
<p>Although Outposts servers integrate seamlessly with the AWS cloud, there are some unique networking considerations when deploying in your data center environment. Amazon EC2 instances on the Outposts server can route traffic over the AWS global network, but you can also enable Local Network Interfaces (LNIs) to directly access your on-premises networks.</p> 
<p>In this post we’ve demonstrated using user-data scripts during instance launch to automate hybrid cloud networking flows tailored to your requirements. With proper planning, you can use the benefits of consistent AWS services and tooling while maintaining connectivity to your existing on-premises infrastructure.</p> 
<p>Ready to get started with hybrid cloud networking on Outposts servers? Check out the <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/what-is-outposts.html" rel="noopener" target="_blank">Outposts server documentation and best practices guide</a> to begin planning your on-premises deployment.</p>
