---
title: "Scaling AWS Outposts rack deployments with ACE racks"
url: "https://aws.amazon.com/blogs/compute/scaling-aws-outposts-rack-deployments-with-ace-racks/"
date: "Tue, 13 Dec 2022 21:36:05 +0000"
author: "Sheila Busser"
feed_url: "https://aws.amazon.com/blogs/compute/tag/aws-outposts/feed/"
---
<p><em>This blog post is written by Eric Vasquez, Specialist Hybrid Edge Solutions Architect, and Paul Scherer, Senior Network Service Tech.</em></p> 
<h2>Overview</h2> 
<p><a href="https://aws.amazon.com/outposts/">AWS Outposts</a> brings managed, monitored AWS infrastructure, compute, and storage to your on-premises environment. It provides the same AWS APIs, and console experience you would get within the AWS Region to which the Outpost is homed to. You may already have an Outposts rack. An Outpost can consist of one or more racks creating a pool of consumable resources as a single logical Outpost. In this post, we will introduce you to an Aggregation, Core, Edge (ACE) rack.</p> 
<p>Depending on your familiarity with the Outpost family, you might have already heard about an ACE rack. An ACE rack serves as an aggregation point for multi-rack Outpost deployments. ACE racks reduce the physical networking port requirements as well as the logical interfaces needed, while allowing for connectivity between multiple racks in your logical Outpost. ACE racks are recommended for customers with planned deployments beyond three racks excluding the ACE rack itself.</p> 
<p>We recommend that all customers leverage an ACE rack if planning expansions beyond three racks in the long-term, even if the initial deployment is a single rack. An ACE rack contains four routers, and these routers can connect to either two or four customer upstream devices. For the best redundancy, reliability, and resiliency, we recommend deploying an ACE rack to four upstream customer devices.</p> 
<p>ACE racks support 10G, 40G, and 100G connections to a customer network. However, 100G connections between each ACE router to a customer device are recommended.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/08/06/Ace_Rack_Updated.png"><img alt="" class="wp-image-22608 size-full aligncenter" height="554" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/08/06/Ace_Rack_Updated.png" width="1034" /></a>Outpost extension from the Region and ACE rack deployment in a 15-rack Outpost configuration</p> 
<p>Each Outposts rack comes standard with redundant Outpost networking devices, power supplies, and two top-of-rack patch panels which serve as demarcation points between the Outpost rack and your customer networking device (CND). For the remainder of this post, we’ll refer to the Outpost Networking Devices as OND and customer switches/routers as CND. The Outpost rack ONDs form Border Gateway Protocol (BGP) neighbor relationships with either your CND or the ACE rack using point-to-point (P2P) Virtual LAN (VLAN) interfaces.</p> 
<p>For Outposts installation without an ACE rack, each Outposts OND connects to your LAN using single-mode or multi-mode fiber with LC connectors supporting 1G, 10G, 40G, or 100G connectivity. We provide flexibility for the CNDs and allow either Layer 2 or Layer 3 devices, including firewalls. Each OND uses a single LACP port channel that carries 2 VLAN point-to-points virtual interfaced (VIF)to establish 2 BGP relationships over the port channel to your upstream CND and aggregate total bandwidth. This results in each Outpost rack requiring a minimum of two physical uplinks, but as a general best practice we recommend two-per-device for a total of four uplinks, along with two LACP port channels and 4 <a href="https://www.rfc-editor.org/rfc/rfc4675.txt">VLAN</a> to establish point-to-points (P2P) BGP peering’s. Note that the IP’s used in the following diagram are just examples.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/29/LAG-and-VLAN.png"><img alt="Outpost Service link and Local Gateway VLAN" class="aligncenter size-full wp-image-19515" height="254" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/29/LAG-and-VLAN.png" width="824" /></a>Outpost service link and local gateway VLAN</p> 
<p>As we continue to expand rack deployments, so will the number of physical uplinks and VLAN interfaces required for the added OND to a CND. When we introduce the ACE rack, the OND is no longer attached to your CND. Instead, it goes directly to ACE devices, which provide at least one uplink to your network switch/router. In this topology, AWS owns the VLAN interface allocation and configuration between compute rack OND and the ACE routers.</p> 
<p>Let’s cover the potential downsides to a multi-rack installation without an ACE rack. In this case, we have a three-rack Outpost deployment, with one uplink (two per rack) from each rack OND to the CND. This would require you to provide: six physical ports on your devices, six fiber cables,12 VLAN VIFs, 12 P2P subnets potentially exhausting 24 IPs, and six port channels.</p> 
<p>In comparison to a three-rack install that sits behind an ACE rack, you provide fewer physical network ports on your devices, fewer fiber cabling uplinks, fewer VLAN VIFs, fewer port channels, and fewer P2P’s. Each ACE router will have its own LACP port channel with 2x VLAN VIFs in each channel (the same as an Outposts Networking Devices (OND)&nbsp;&lt;&gt; Customer connection). The following table highlights the advantages in using an ACE rack when running a multi-rack Outpost, which becomes more desirable as you continue to scale.</p> 
<table border=".15" style="height: 566px;" width="664"> 
 <tbody> 
  <tr> 
   <td width="111"></td> 
   <td colspan="2" style="background-color: #ded9d9; width: 167px;" width="167"> <p style="text-align: center;"><strong>2-Rack Outpost</strong></p> <p style="text-align: center;"><strong>Installation</strong></p> </td> 
   <td colspan="2" style="background-color: #ded9d9; width: 167px; text-align: center;" width="167"> <p><strong>3-Rack Outpost</strong></p> <p><strong>Installation</strong></p></td> 
   <td colspan="2" style="background-color: #ded9d9; width: 156px;" width="156"> <p style="text-align: center;"><strong>4-Rack Outpost</strong></p> <p style="text-align: center;"><strong>Installation</strong></p> </td> 
  </tr> 
  <tr> 
   <td width="111"> <p style="text-align: center;"><strong>Requirement</strong></p> <p style="text-align: center;"></p> </td> 
   <td style="text-align: center;" width="90"><strong>Without ACE</strong></td> 
   <td style="text-align: center;" width="77"><strong>With </strong><strong>ACE</strong></td> 
   <td style="text-align: center;" width="90"><strong>Without </strong><strong>ACE</strong></td> 
   <td style="text-align: center;" width="77"><strong>With </strong><strong>ACE</strong></td> 
   <td style="width: 85px; text-align: center;" width="83"><strong>Without ACE</strong></td> 
   <td style="text-align: center;" width="73"><strong>With </strong><strong>ACE</strong></td> 
  </tr> 
  <tr> 
   <td width="111"> <p style="text-align: center;"><strong>Physical Ports</strong></p> </td> 
   <td width="90"> <p style="text-align: center;">4</p> </td> 
   <td width="77"> <p style="text-align: center;">4</p> </td> 
   <td width="90"> <p style="text-align: center;">6</p> </td> 
   <td style="text-align: center;" width="77">4</td> 
   <td width="83"> <p style="text-align: center;">8</p> </td> 
   <td width="73"> <p style="text-align: center;">4</p> </td> 
  </tr> 
  <tr> 
   <td width="111"> <p style="text-align: center;"><strong>Fiber Cables</strong></p> </td> 
   <td width="90"> <p style="text-align: center;">4</p> </td> 
   <td style="text-align: center;" width="77">4</td> 
   <td style="text-align: center;" width="90">6</td> 
   <td style="text-align: center;" width="77">4</td> 
   <td style="text-align: center;" width="83">8</td> 
   <td width="73"> <p style="text-align: center;">4</p> </td> 
  </tr> 
  <tr> 
   <td width="111"> <p style="text-align: center;"><strong>LACP Port Channels</strong></p> </td> 
   <td width="90"> <p style="text-align: center;">4</p> </td> 
   <td style="text-align: center;" width="77">4</td> 
   <td style="text-align: center;" width="90">6</td> 
   <td style="text-align: center;" width="77">4</td> 
   <td width="83"> <p style="text-align: center;">8</p> </td> 
   <td width="73"> <p style="text-align: center;">4</p> </td> 
  </tr> 
  <tr> 
   <td width="111"> <p style="text-align: center;"><strong>VLAN VIFs</strong></p> </td> 
   <td width="90"> <p style="text-align: center;">8</p> </td> 
   <td style="text-align: center;" width="77">8</td> 
   <td style="text-align: center;" width="90">12</td> 
   <td style="text-align: center;" width="77">8</td> 
   <td style="text-align: center;" width="83">16</td> 
   <td width="73"> <p style="text-align: center;">8</p> </td> 
  </tr> 
  <tr> 
   <td style="text-align: center;" width="111"><strong>P2P Subnets</strong></td> 
   <td style="text-align: center;" width="90">8</td> 
   <td style="text-align: center;" width="77">8</td> 
   <td style="text-align: center;" width="90">12</td> 
   <td style="text-align: center;" width="77">8</td> 
   <td style="text-align: center;" width="83">16</td> 
   <td width="73"> <p style="text-align: center;">8</p> </td> 
  </tr> 
 </tbody> 
</table> 
<h2>ACE vs. Non-ACE Rack Components Comparison</h2> 
<p>Furthermore, you should consider the additional weight and power requirements that an ACE rack introduces when planning for multi-rack deployments. In addition to initial kVA requirements for the Outpost racks, you must account for the resources required for an ACE rack. An ACE rack consumes up to 10kVA of power and weighs up to 705 lbs. Carefully planning additional capacity for these resources with your AWS account team will be critical for a successful deployment.</p> 
<p>Similar to an Outpost rack, an ACE rack deployment is monitored by AWS. The rack provides telemetry data transmitted over a set of VPN tunnels back to the anchor points in the Region to which the Outpost is homed. This allows AWS to monitor the rack for hardware failures, performance degradation, and other alarm conditions including Links, Interfaces going down, and BGP drops.</p> 
<p>As part of the Outpost ordering process, AWS will work closely with you to determine the location for install, power availability onsite, and the network configuration of both the Outposts rack and ACE rack. This includes BGP configuration, and the Customer Owned IP Address (<a href="https://docs.aws.amazon.com/outposts/latest/userguide/routing.html">CoIP</a>), which is the pool of IP addresses for route advertisements back to your CND. The CoIP pool allows resources inside your Outpost rack to communicate with on-premises resources and vice-versa. Another connectivity option would be the Direct VPC Routing (<a href="https://aws.amazon.com/blogs/aws/new-direct-vpc-routing-between-on-premises-networks-and-aws-outposts-rack/">DVR</a>) where we advertise VPC subnets associated with your LGW to your on-premises networks. Outposts uses a networking connectivity back to the Region for management purposes called the service link (SL). The SL is an encrypted set of VPN connections used whenever the Outpost communicates with your chosen home Region.</p> 
<h2>Conclusion</h2> 
<p>This post addresses the most common questions surrounding ACE racks, how an ACE rack can be deployed, and why an ACE rack would be leveraged for a multi-Outpost rack deployment. In this post, we demonstrated how an ACE rack serves as a consolidation point in your on-premises environment, making multi-rack deployments scalable, while reducing complexity and physical port allocation for connectivity between an Outpost and your LAN. In addition, we described how you can get this process started. If you want to learn more about Outposts fundamentals and how you can build your applications with AWS services using Outposts for hybrid cloud deployments, you can check out the <a href="https://docs.aws.amazon.com/outposts/latest/userguide/get-started-outposts.html">Outposts user guide</a>.</p>
