---
title: "Implementing backup for workloads running on AWS Outposts servers"
url: "https://aws.amazon.com/blogs/compute/implementing-backup-for-workloads-running-on-aws-outposts-servers/"
date: "Thu, 12 Dec 2024 16:54:48 +0000"
author: "aostan"
feed_url: "https://aws.amazon.com/blogs/compute/tag/aws-outposts/feed/"
---
<p><em>This post is written by Leonardo Queirolo, Senior Cloud Support Engineer and Tareq Rajabi, Senior Solutions Architect, Hybrid Cloud</em></p> 
<p><a href="https://aws.amazon.com/outposts/servers/" rel="noopener" target="_blank">AWS Outposts servers</a> provide fully managed AWS infrastructure, services, APIs, and tools to on-premises and edge locations with limited space or small capacity requirements, such as retail stores, branch offices, healthcare provider locations, or factory floors. Outposts servers provide local compute and networking services.</p> 
<p>Outposts servers come with internal NVMe SSD instance storage, supporting local storage used for data access and processing on premises, and for launching <a href="https://aws.amazon.com/ebs/" rel="noopener" target="_blank">Amazon Elastic Block Store (Amazon EBS</a>)-backed <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html" rel="noopener" target="_blank">Amazon Machine Images (AMIs).</a> The data on these volumes persists after an instance reboot but does not persist after an instance termination. In order for data to persist beyond the lifetime of the instance, it is important to back up your data to a persistent AWS storage, such as an <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket or an <a href="https://aws.amazon.com/ebs/" rel="noopener" target="_blank">Amazon Elastic Block Store&nbsp;(Amazon EBS)</a> volume.</p> 
<p>In this post, we explore several approaches to back up the data stored in the instance storage volumes of your EC2 instances running on an Outposts server to a persistent storage solution from AWS, and explore their benefits and use cases.</p> 
<h2>Planning for failure</h2> 
<p>When evaluating a backup strategy, it’s important to understand the failure modes you are looking to recover from. Some examples are ransomware attacks, accidental data deletion, hardware failure, or a wide scale issue impacting the whole facility where your Outposts servers and on-premises devices (such as network switches, storage appliances) reside. These failures come in many forms and are often unplanned and unexpected events. Next, understand what is considered acceptable recovery for your business. For example, what are the <a href="https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html" rel="noopener" target="_blank">Recovery Time Objective (RTO) and Recovery Point Objective (RPO)</a> for your workload running on Outposts servers? These two values, defined by your organization, profile how long a service can be down during recovery and quantify the acceptable amount of data loss, helping you define the appropriate backup strategy.</p> 
<h2>Scenario 1: Backup to AWS storage in an AWS Region</h2> 
<p>Backup to an <a href="https://aws.amazon.com/about-aws/global-infrastructure/regions_az/" rel="noopener" target="_blank">AWS Region</a> enables data redundancy outside of the data center or facility where your Outpost resides, taking advantage of the durability, high availability, and scalability provided natively by the storage in the Region. This approach offers flexibility for restoration to the Region or to an Outposts server in a different edge location if the original data center/facility is impacted by an irrecoverable incident. However, when restoring the data back to an Outposts server, this approach could result in relatively high RTO, depending on the throughput of the <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/region-connectivity.html" rel="noopener" target="_blank">service link</a> and the amount of data to restore. In the following sections, we will cover using the <a href="https://aws.amazon.com/disaster-recovery/" rel="noopener" target="_blank">AWS Elastic Disaster Recovery (AWS DRS)</a> and an open source solution based on operating system tools and <a href="https://aws.amazon.com/systems-manager/" rel="noopener" target="_blank">AWS Systems Manager (AWS SSM)</a>.</p> 
<h2>Option 1: <a href="https://aws.amazon.com/disaster-recovery/" rel="noopener" target="_blank">AWS Elastic Disaster Recovery (AWS DRS)</a></h2> 
<p>You can use AWS DRS to perform a continuous replication of workloads that reside on the Outposts C6id server powered by Intel processors (C6gd are not supported, since <a href="https://docs.aws.amazon.com/drs/latest/userguide/Supported-Operating-Systems.html" rel="noopener" target="_blank">only 64-bit operating systems built for the x86 system architecture are supported by AWS DRS</a>) to a <a href="https://docs.aws.amazon.com/drs/latest/userguide/Network-Settings-Preparations.html#Staging-Area" rel="noopener" target="_blank">staging area subnet</a> in the Region. AWS DRS provides nearly continuous, block-level replication in the Region and creates periodic EBS Snapshots according to the <a href="https://docs.aws.amazon.com/drs/latest/userguide/How-Many-Snapshots.html" rel="noopener" target="_blank">Point in Time (PIT) state schedule for AWS DRS</a>.</p> 
<p>The following diagram shows the continuous replication of the data in the instance store volumes through AWS DRS. The PIT EBS Snapshots are used to create Amazon EBS-backed AMIs as a backup of the EC2 instances running on the Outposts server.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/figure-1.jpg"><img alt="Figure 1 - Continuous replication of the Instance Store Volumes data from the instances running Outpost Server to a staging area in the parent region through DRS" class="size-full wp-image-23123 aligncenter" height="551" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/figure-1.jpg" width="1031" /></a></p> 
<p style="text-align: center;"><em>Figure 1 – Continuous replication of the Instance Store Volumes data from the instances running Outpost Server to a staging area in the parent region through DRS</em></p> 
<p>Despite AWS DRS not supporting the failback from the Region to Outposts servers, you can use the EBS snapshots taken by AWS DRS to restore the data back to the Outposts server at the desired PIT following the steps described in this post.</p> 
<h3>Prerequisites</h3> 
<p>The following prerequisites are required to complete the walkthrough:</p> 
<ol> 
 <li>The EC2 instance to restore running on the Outposts server has been <a href="https://docs.aws.amazon.com/drs/latest/userguide/adding-servers.html" rel="noopener" target="_blank">added as a source server</a> to AWS DRS by <a href="https://docs.aws.amazon.com/drs/latest/userguide/agent-installation.html" rel="noopener" target="_blank">installing the AWS Replication Agent</a>.</li> 
 <li>The initial sync has been completed and the <a href="https://docs.aws.amazon.com/drs/latest/userguide/recovery-dashboard.html#data-replication-stat" rel="noopener" target="_blank">data replication status</a> is showing as healthy.</li> 
</ol> 
<h3>Restore the entire EC2 instance on the same or a different Outposts server</h3> 
<ol> 
 <li>Use the <a href="https://docs.aws.amazon.com/cli/latest/reference/drs/describe-recovery-snapshots.html" rel="noopener" target="_blank">describe-recovery-snapshots</a> command to list the PIT Snapshots taken by AWS DRS for the source server to restore.<code>$ aws drs describe-recovery-snapshots --source-server &lt;source-server-id&gt;</code></li> 
</ol> 
<p style="padding-left: 40px;">2. Based on the time in which you want to restore your data, retrieve the corresponding EBS Snapshots in the output of the command. The following is an example of the output:</p> 
<pre><code class="lang-json">{
    "items": [
       {
            "ebsSnapshots": [
                "snap-07bf348d58151a432"
            ],
            "expectedTimestamp": "2024-06-13T16:40:00+00:00",
            "snapshotID": "pit-a4877ff6fa68561bf",
            "sourceServerID": "s-a080ceb10af7275a7",
            "timestamp": "2024-06-13T16:46:56.645979+00:00"
        },
        {
            "ebsSnapshots": [
                "snap-0496020ff7f83486d"
            ],
            "expectedTimestamp": "2024-06-13T16:30:00+00:00",
            "snapshotID": "pit-aece827519e1b0fbb",
            "sourceServerID": "s-a080ceb10af7275a7",
            "timestamp": "2024-06-13T16:37:06.600323+00:00"
        },
        {
            "ebsSnapshots": [
                "snap-0d7ebd23e56346cea"
            ],
            "expectedTimestamp": "2024-06-13T16:20:00+00:00",
            "snapshotID": "pit-a56960f89ff12579e",
            "sourceServerID": "s-a080ceb10af7275a7",
            "timestamp": "2024-06-13T16:27:01.595791+00:00"
        },
…
…</code></pre> 
<p style="padding-left: 40px;">3. Open the <a href="https://console.aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon EC2 console</a>. In the navigation pane, choose <strong>Snapshots</strong> and filter by the <strong>Snapshot ID</strong> chosen in the previous step: snap-07bf348d58151a432.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/2-5.png"><img alt="" class="size-full wp-image-23122 aligncenter" height="288" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/2-5.png" width="1363" /></a></p> 
<p style="padding-left: 40px;">4. Choose <strong>Actions</strong>, <strong>Create image from snapshot, </strong>and specify the<strong> Image name. </strong>You can leave the other information as default or customize as desired.</p> 
<p style="padding-left: 40px;">5. To perform the restore, <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/launch-instance.html#launch-instances" rel="noopener" target="_blank">launch a new EC2 instance on the same or a different Outposts server</a> from the Amazon EBS-backed AMI created in the previous step.</p> 
<p>Note that since AMIs are downloaded from the Region with every instance launch on Outposts servers, this approach could result in an RTO spanning hours, depending on the throughput of the <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/region-connectivity.html" rel="noopener" target="_blank">service link</a> and the size of the local instance storage from which the Snapshot and AMI were taken by AWS DRS. Alternatively, if you need to restore only some files and directories, you can do so by launching the EC2 instance in the Region from the AMI taken in Step 4 and then transferring the desired data from that instance to the source server running on Outposts.</p> 
<h3>Option 2: Backup to the Region using an open source solution</h3> 
<p>In addition to AWS DRS, you can use open source solutions and/or operating system (OS) functions to back up data from local instance storage to a Region. Consider this approach when you want a highly-customizable solution for workloads where lack of commercial support is acceptable. The <a href="https://github.com/aws-samples/backup-outposts-servers-linux-instance" rel="noopener" target="_blank">open source solution</a> uses <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html" rel="noopener" target="_blank">AWS Systems Manager Automation</a> and OS functions to take an Amazon EBS-backed AMI in the Region from a Linux EC2 instance running on your Outposts server. The following diagram provides a high-level overview of the solution.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/3.jpg"><img alt="Figure 2 – Wokflow of the open source solution" class="size-full wp-image-23121 aligncenter" height="721" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/3.jpg" width="1099" /></a></p> 
<p style="text-align: center;"><em>Figure 2 – Workflow of the open source solution</em></p> 
<ol> 
 <li>The Automation creates a helper instance and a baseline EBS volume attached to it in the Region, using an <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a></li> 
 <li>The Automation executes commands on the OS of the EC2 instance running on the Outposts server to perform preliminary checks and start syncing data from the local instance store volume to the baseline EBS volume in the Region.</li> 
 <li>The sync continues until the data has been transferred successfully.</li> 
 <li>When the sync completes, the Automation takes an EBS Snapshot of the baseline EBS volume and then creates an Amazon EBS-backed AMI from it.</li> 
</ol> 
<h3>Create the Automation document</h3> 
<ol> 
 <li>Open the github page of the open source solution <a href="https://github.com/aws-samples/backup-outposts-servers-linux-instance" rel="noopener" target="_blank">backup-outposts-servers-linux-instance</a>.</li> 
 <li>Follow the <strong>Installation Instructions </strong>to create the Systems Manager Automation document.</li> 
</ol> 
<h3>Back up an EC2 instance running on Outposts server</h3> 
<ol> 
 <li>After creating the Automation document, follow the <strong>Usage Instructions</strong> to execute the Automation and initiate the backup.</li> 
 <li>Monitor the <strong>Execution status</strong> in the System Manager Automation console.</li> 
</ol> 
<h3>Restore the entire EC2 instance on the same or a different Outposts server</h3> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon EC2 console</a>. In the navigation pane, choose <strong>AMIs</strong> and filter by the <strong>AMI names</strong> that contain the <strong>InstanceId </strong>to restore.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/4-4.png"><img alt="" class="size-full wp-image-23120 aligncenter" height="300" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2024/11/22/4-4.png" width="1348" /></a></p> 
<p style="padding-left: 40px;">2. Select the desired AMI to restore and note its <strong>AMI ID.</strong></p> 
<p style="padding-left: 40px;">3. To perform the restore, <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/launch-instance.html#launch-instances" rel="noopener" target="_blank">launch a new EC2 instance on the same or a different Outposts server</a> from the Amazon EBS-backed AMI identified in the previous step.</p> 
<h2>Considerations for data residency and service link bandwidth</h2> 
<p>Data residency is a critical consideration for organizations that need to collect and store data in their own data centers for regulatory or compliance reasons. In this case, users cannot back up their data to the Region and need to consider backing up to another on-premises system.</p> 
<p>Another consideration is the impact on the service link connectivity when performing backup and restore operations between the Outposts and the Region. When implementing the solutions described in the “Backup to AWS storage in an AWS Region” scenario, both your backup/restore and management/monitoring operations for your Outpost rely on the service link connectivity. Although AWS DRS provides block-level replication, the open source solution we discuss in this post only replicates data, resulting in smaller snapshot sizes for users with lower service link bandwidth requirement.</p> 
<p>If you foresee bandwidth constraints for your service link, consider backing up to another on-premises system that is reachable through the <a href="https://docs.aws.amazon.com/outposts/latest/server-userguide/local-network-interface.html" rel="noopener" target="_blank">local network interface</a> (LNI) of your Outposts server.</p> 
<h2>Scenario 2: Backup to AWS storage in your on-premises environment</h2> 
<p>For the preceding reasons, you may need to back up your workload running on Outposts server to a persistent AWS storage system within the same geo political boundary. To do so, you can use an <a href="https://aws.amazon.com/outposts/rack/" rel="noopener" target="_blank">AWS Outposts rack</a> that resides in the same or a different physical location and is reachable through the LNI of your Outposts server.</p> 
<p><a href="https://aws.amazon.com/outposts/rack/" rel="noopener" target="_blank">Outposts rack</a> with <a href="https://aws.amazon.com/s3/outposts/" rel="noopener" target="_blank">Amazon S3 on Outposts</a> allows you to run AWS infrastructure, services, and object storage to your on-premises to meet local data processing and data residency needs while offering the AWS durable storage that can be used to store your backup.</p> 
<p>Thanks to this, you can use the same approaches described in the “Backup to AWS storage in an AWS Region” section at a high level to back up your data, while the storage is hosted on the Outposts rack. When evaluating this approach, keep in mind these important <a href="https://docs.aws.amazon.com/ebs/latest/userguide/snapshots-outposts.html#considerations" rel="noopener" target="_blank">considerations for local snapshots</a>.</p> 
<p>With this approach, you can store your backup on premises to meet your data residency requirements. This also keeps the network traffic for your backup and restore within your on-premises network, without impacting the service link.</p> 
<h2>Conclusion</h2> 
<p>In this post, we showed different approaches to design backup and restore strategies for your workloads running on Outposts servers. Implementing the right approach can help protect your organization’s data against loss or corruption while meeting your performance, RTO, RPO, and data residency needs, with backup destinations ranging from AWS storage in the Region, locally on Outposts rack, or in a hybrid architecture.</p>
