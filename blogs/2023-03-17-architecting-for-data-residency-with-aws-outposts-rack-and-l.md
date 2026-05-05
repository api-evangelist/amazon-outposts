---
title: "Architecting for data residency with AWS Outposts rack and landing zone guardrails"
url: "https://aws.amazon.com/blogs/compute/architecting-for-data-residency-with-aws-outposts-rack-and-landing-zone-guardrails/"
date: "Fri, 17 Mar 2023 18:24:42 +0000"
author: "Sheila Busser"
feed_url: "https://aws.amazon.com/blogs/compute/tag/aws-outposts/feed/"
---
<p><em>This blog post was written by Abeer Naffa’, <span class="optional-wrapper">Sr. Solutions Architect, Solutions Builder AWS</span>, David Filiatrault, <span class="optional-wrapper">Principal Security Consultant</span><span class="optional-wrapper">, AWS</span> and Jared Thompson, <span class="optional-wrapper">Hybrid Edge SA Specialist</span><span class="optional-wrapper">, AWS</span>.<br /> </em></p> 
<p>In this post, we will explore how organizations can use <a href="https://docs.aws.amazon.com/controltower/latest/userguide/planning-your-deployment.html">AWS Control Tower landing zone</a> and<a href="https://aws.amazon.com/organizations/"> AWS Organizations</a> custom guardrails to enable compliance with data residency requirements on <a href="https://aws.amazon.com/outposts/rack/?nc=sn&amp;loc=2">AWS Outposts rack</a>. We will discuss how custom guardrails can be leveraged to limit the ability to store, process, and access data and remain isolated in specific geographic locations, how they can be used to enforce security and compliance controls, as well as, which prerequisites organizations should consider before implementing these guardrails.</p> 
<p>Data residency is a critical consideration for organizations that collect and store sensitive information, such as Personal Identifiable Information (PII), financial, and healthcare data. With the rise of cloud computing and the global nature of the internet, it can be challenging for organizations to make sure that their data is being stored and processed in compliance with local laws and regulations.</p> 
<p>One potential solution for addressing <a href="https://d1.awsstatic.com/product-marketing/Outposts/AWS%20Outposts%20Data%20Residency%20eBook.pdf">data residency</a> challenges with AWS is to use Outposts rack, which allows organizations to run AWS infrastructure on premises and in their own data centers. This lets organizations store and process data in a location of their choosing. An Outpost is seamlessly connected to an <a href="https://aws.amazon.com/about-aws/global-infrastructure/regions_az/">AWS Region</a> where it has access to the full suite of AWS services managed from a single plane of glass, the <a href="https://aws.amazon.com/console/">AWS Management Console</a> or the AWS Command Line Interface (<a href="https://aws.amazon.com/cli/">AWS CLI</a>).&nbsp; Outposts rack can be configured to utilize landing zone to further adhere to data residency requirements.</p> 
<p>The landing zones are a set of tools and best practices that help organizations establish a secure and compliant multi-account structure within a cloud provider. A landing zone can also include Organizations to set policies – guardrails – at the root level, known as <a href="https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html">Service Control Policies (SCPs</a>) across all member accounts. This can be configured to enforce certain data residency requirements.</p> 
<p>When leveraging Outposts rack to meet data residency requirements, it is crucial to have control over the in-scope data movement from the Outposts. This can be accomplished by implementing landing zone best practices and the suggested guardrails. The main focus of this blog post is on the custom policies that restrict data snapshots, prohibit data creation within the Region, and limit data transfer to the Region.</p> 
<h2>Prerequisites</h2> 
<p>Landing zone best practices and custom guardrails can help when data needs to remain in a specific locality where the Outposts rack is also located.&nbsp; This can be completed by defining and enforcing policies for data storage and usage within the landing zone organization that you set up. The following prerequisites should be considered before implementing the suggested guardrails:</p> 
<p style="padding-left: 23px;">1. <strong>AWS Outposts rack</strong></p> 
<p style="text-align: left; padding-left: 40px;">AWS has installed your Outpost and handed off to you. An Outpost may comprise of one or more racks connected together at the site. This means that you can start using AWS services on the Outpost, and you can manage the Outposts rack using the same tools and interfaces that you use in AWS Regions.</p> 
<p style="padding-left: 23px;">2.&nbsp;<strong>AWS Control Tower landing zone</strong></p> 
<p style="padding-left: 40px;">Control Tower is a managed service that provides a pre-packaged set of best-practice blueprints for setting up and governing multi-account AWS environments. You’ll need to have Control Tower fully implemented in your environment before you can deploy custom guardrails.</p> 
<p style="padding-left: 40px;">Control Tower launches a key resource associated with your account, called a landing zone, which serves as a home for your organizations and their accounts.</p> 
<p style="padding-left: 40px;">Note that Control Tower relies on Organizations to create and manage multi-account structures. You’ll need to have Organizations set up and configured as well.</p> 
<p style="padding-left: 23px;">3. <strong>Set up the data residency guardrails</strong></p> 
<p style="padding-left: 40px;">Using Organizations, you must make sure that the Outpost is ordered within a workload account in the landing zone.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2023/03/14/Figure-1-Landing-Zone-Accelerator-Outposts-workload-on-AWS-high-level-Architecture.png"><img alt="Figure 1 Landing Zone Accelerator Outposts workload on AWS high level Architecture" class="aligncenter size-full wp-image-20149" height="331" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2023/03/14/Figure-1-Landing-Zone-Accelerator-Outposts-workload-on-AWS-high-level-Architecture.png" width="771" /></a></p> 
<p style="text-align: center;">Figure 1: Landing Zone Accelerator – Outposts workload on AWS high level Architecture</p> 
<h2><strong>Utilizing Outposts rack for regulated components</strong></h2> 
<p>When local regulations require regulated workloads to stay within a specific boundary, or when an AWS Region or <a href="https://aws.amazon.com/about-aws/global-infrastructure/localzones/">AWS Local Zone</a> isn’t available in your jurisdiction, you can still choose to host your regulated workloads on Outposts rack for a consistent cloud experience. When opting for Outposts rack, note that, as part of the <a href="https://docs.aws.amazon.com/outposts/latest/userguide/security.html">shared responsibility model</a>, <a href="https://aws.amazon.com/outposts/rack/faqs/#Security_.26_compliance">customers are responsible</a> for attesting to physical security, access controls, and <a href="https://docs.aws.amazon.com/outposts/latest/userguide/compliance-validation.html">compliance validation</a> regarding the Outposts, as well as, environmental requirements for the facility, networking, and power. Utilizing Outposts rack requires that you procure and manage the data center within the city, state, province, or country boundary for your applications’ regulated components, as required by local regulations.</p> 
<p>Procuring two or more racks in the diverse data centers can help with the <a href="https://docs.aws.amazon.com/pdfs/whitepapers/latest/aws-outposts-high-availability-design/aws-outposts-high-availability-design.pdf#aws-outposts-high-availability-design">high availability</a> for your workloads. This is because it provides redundancy in case of a single rack or server failure. Additionally, having redundant network paths between Outposts rack and the parent Region can help make sure that your application remains connected and continue to operate even if one network path fails.</p> 
<p>However, for regulated workloads with strict service level agreements (SLA), you may choose to spread Outposts racks across two or more isolated data centers within regulated boundaries. This helps make sure that your data remains within the designated geographical location and meets local data residency requirements.</p> 
<p>In this post, we consider a scenario with one data center, but consider the specific requirements of your workloads and the regulations that apply to determine the most appropriate <a href="https://docs.aws.amazon.com/pdfs/whitepapers/latest/aws-outposts-high-availability-design/aws-outposts-high-availability-design.pdf#aws-outposts-high-availability-design">high availability</a> configurations for your case.</p> 
<h2>Outposts rack workload data residency guardrails</h2> 
<p>Organizations provide central governance and management for multiple accounts. Central security administrators use SCPs with Organizations to establish controls to which all <a href="https://aws.amazon.com/iam/">AWS Identity and Access Management (IAM)</a> principals (users and roles) adhere.</p> 
<p>Now, you can use SCPs to set permission guardrails.&nbsp; A suggested preventative controls for <strong>data residency on Outposts rack</strong> that leverage the implementation of SCPs are shown as follows. SCPs enable you to set permission guardrails by defining the maximum available permissions for IAM entities in an account. If an SCP denies an action for an account, then none of the entities in the account can take that action, even if their IAM permissions let them. The guardrails set in SCPs apply to all IAM entities in the account, which include all users, roles, and the account root user.</p> 
<p>Upon finalizing these prerequisites, you can <a href="https://aws.amazon.com/blogs/security/how-to-use-service-control-policies-to-set-permission-guardrails-across-accounts-in-your-aws-organization/">create the guardrails</a> for the Outposts Organization Unit (OU).</p> 
<table border=".15" style="height: 118px;" width="743"> 
 <tbody> 
  <tr> 
   <td style="padding-left: 40px;" width="527">Note that while the following guidelines serve as helpful guardrails – SCPs – for data residency, you should consult internally with legal and security teams for specific organizational requirements.</td> 
  </tr> 
 </tbody> 
</table> 
<p><strong>&nbsp;</strong>To exercise better control over workloads in the Outposts rack and prevent data transfer from Outposts to the Region or data storage outside the Outposts, consider implementing the following guardrails. Additionally, local regulations may dictate that you set up these additional guardrails.</p> 
<ol> 
 <li>When your data residency requirements require restricting data transfer/saving to the Region, consider the following guardrails:</li> 
</ol> 
<p style="padding-left: 80px;">a. Deny copying data from Outposts to the Region for <a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud (Amazon EC2</a>), <a href="https://aws.amazon.com/rds/">Amazon Relational Database Service (Amazon RDS</a>), <a href="https://aws.amazon.com/elasticache/">Amazon ElastiCache</a> and data sync “DenyCopyToRegion”.</p> 
<p style="padding-left: 80px;">b. Deny <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service (Amazon S3</a>) put action to the Region “DenyPutObjectToRegionalBuckets”.</p> 
<table border=".15" style="height: 161px;" width="734"> 
 <tbody> 
  <tr> 
   <td width="527"> <p style="padding-left: 40px;">If your data residency requirements mandate restrictions on data storage in the Region,&nbsp; consider implementing this guardrail to prevent&nbsp; the use of S3 in the Region.</p> <p style="padding-left: 40px;">Note: You can use <a href="https://aws.amazon.com/s3/outposts/">Amazon S3 for Outposts</a>.</p> </td> 
  </tr> 
 </tbody> 
</table> 
<p style="padding-left: 80px;">c. If your data residency requirements mandate restrictions on data storage in the Region, consider implementing “DenyDirectTransferToRegion” guardrail.</p> 
<table border=".15" style="height: 74px;" width="734"> 
 <tbody> 
  <tr> 
   <td width="623"> <p style="padding-left: 40px;">Out of Scope is metadata such as tags, or operational data such as KMS keys.</p> </td> 
  </tr> 
 </tbody> 
</table> 
<pre><code class="lang-json">{
  "Version": "2012-10-17",
  "Statement": [
      {
      "Sid": "DenyCopyToRegion",
      "Action": [
        "ec2:ModifyImageAttribute",
        "ec2:CopyImage",  
        "ec2:CreateImage",
        "ec2:CreateInstanceExportTask",
        "ec2:ExportImage",
        "ec2:ImportImage",
        "ec2:ImportInstance",
        "ec2:ImportSnapshot",
        "ec2:ImportVolume",
        "rds:CreateDBSnapshot",
        "rds:CreateDBClusterSnapshot",
        "rds:ModifyDBSnapshotAttribute",
        "elasticache:CreateSnapshot",
        "elasticache:CopySnapshot",
        "datasync:Create*",
        "datasync:Update*"
      ],
      "Resource": "*",
      "Effect": "Deny"
    },
    {
      "Sid": "DenyDirectTransferToRegion",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:CreateTable",
        "ec2:CreateTrafficMirrorTarget",
        "ec2:CreateTrafficMirrorSession",
        "rds:CreateGlobalCluster",
        "es:Create*",
        "elasticfilesystem:C*",
        "elasticfilesystem:Put*",
        "storagegateway:Create*",
        "neptune-db:connect",
        "glue:CreateDevEndpoint",
        "glue:UpdateDevEndpoint",
        "datapipeline:CreatePipeline",
        "datapipeline:PutPipelineDefinition",
        "sagemaker:CreateAutoMLJob",
        "sagemaker:CreateData*",
        "sagemaker:CreateCode*",
        "sagemaker:CreateEndpoint",
        "sagemaker:CreateDomain",
        "sagemaker:CreateEdgePackagingJob",
        "sagemaker:CreateNotebookInstance",
        "sagemaker:CreateProcessingJob",
        "sagemaker:CreateModel*",
        "sagemaker:CreateTra*",
        "sagemaker:Update*",
        "redshift:CreateCluster*",
        "ses:Send*",
        "ses:Create*",
        "sqs:Create*",
        "sqs:Send*",
        "mq:Create*",
        "cloudfront:Create*",
        "cloudfront:Update*",
        "ecr:Put*",
        "ecr:Create*",
        "ecr:Upload*",
        "ram:AcceptResourceShareInvitation"
      ],
      "Resource": "*",
      "Effect": "Deny"
    },
    {
      "Sid": "DenyPutObjectToRegionalBuckets",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": ["arn:aws:s3:::*"],
      "Effect": "Deny"
    }
  ]
}
</code></pre> 
<ol start="2"> 
 <li>If your data residency requirements require limitations on data storage in the Region, consider implementing this guardrail “DenySnapshotsToRegion” and “DenySnapshotsNotOutposts” to restrict the use of snapshots in the Region.</li> 
</ol> 
<p style="padding-left: 80px;">a. Deny creating snapshots of your Outpost data in the Region “DenySnapshotsToRegion”</p> 
<p style="padding-left: 80px;"><em>&nbsp;Make sure to update the Outposts “</em><strong><em>&lt;</em>outpost_arn_pattern<em>&gt;</em></strong><strong><em>”.</em></strong></p> 
<p style="padding-left: 80px;">b. Deny copying or modifying Outposts Snapshots “DenySnapshotsNotOutposts”</p> 
<p style="padding-left: 80px;"><em>Make sure to update the Outposts “</em><strong><em>&lt;</em>outpost_arn_pattern<em>&gt;</em></strong><em>”.</em></p> 
<p><strong><em>Note</em></strong><em>: “</em><em>&lt;</em><strong>outpost_arn_pattern</strong><em>&gt;” default is </em>arn:aws:outposts:*:*:outpost/*</p> 
<pre><code class="lang-json">{
  "Version": "2012-10-17",
  "Statement": [

    {
      "Sid": "DenySnapshotsToRegion",
      "Effect":"Deny",
      "Action":[
        "ec2:CreateSnapshot",
        "ec2:CreateSnapshots"
      ],
      "Resource":"arn:aws:ec2:*::snapshot/*",
      "Condition":{
         "ArnLike":{
            "ec2:SourceOutpostArn":"&lt;outpost_arn_pattern&gt;"
         },
         "Null":{
            "ec2:OutpostArn":"true"
         }
      }
    },
    {

      "Sid": "DenySnapshotsNotOutposts",          
      "Effect":"Deny",
      "Action":[
        "ec2:CopySnapshot",
        "ec2:ModifySnapshotAttribute"
      ],
      "Resource":"arn:aws:ec2:*::snapshot/*",
      "Condition":{
         "ArnLike":{
            "ec2:OutpostArn":"&lt;outpost_arn_pattern&gt;"
         }
      }
    }

  ]
}
</code></pre> 
<ol start="3"> 
 <li>This guardrail helps to prevent the launch of Amazon EC2 instances or creation of network interfaces in non-Outposts subnets. It is advisable to keep data residency workloads within the Outposts rather than the Region to ensure better control over regulated workloads. This approach can help your organization achieve better control over data residency workloads and improve governance over your AWS Organization.</li> 
</ol> 
<p style="padding-left: 40px;"><em>Make sure to update the Outposts subnets </em><strong>“&lt;outpost_subnet_arns</strong><strong>&gt;”</strong>.</p> 
<pre><code class="lang-json">{
"Version": "2012-10-17",
  "Statement":[{
    "Sid": "DenyNotOutpostSubnet",
    "Effect":"Deny",
    "Action": [
      "ec2:RunInstances",
      "ec2:CreateNetworkInterface"
    ],
    "Resource": [
      "arn:aws:ec2:*:*:network-interface/*"
    ],
    "Condition": {
      "ForAllValues:ArnNotEquals": {
        "ec2:Subnet": ["&lt;outpost_subnet_arns&gt;"]
      }
    }
  }]
}</code></pre> 
<h2>Additional considerations</h2> 
<p>When implementing data residency guardrails on Outposts rack, consider backup and <a href="https://docs.aws.amazon.com/whitepapers/latest/aws-outposts-high-availability-design/storage.html">disaster recovery</a> strategies to make sure that your data is protected in the event of an outage or other unexpected events. This may include creating regular backups of your data, implementing disaster recovery plans and procedures, and using redundancy and failover systems to minimize the impact of any potential disruptions. Additionally, you should make sure that your backup and disaster recovery systems are compliant with any relevant data residency regulations and requirements. You should also test your backup and disaster recovery systems regularly to make sure that they are functioning as intended.</p> 
<p>Additionally, the provided SCPs for Outposts rack in the above example do not block the “logs:PutLogEvents”. Therefore, even if you implemented data residency guardrails on Outpost, the application may log data to CloudWatch logs in the Region.</p> 
<table border=".15" style="height: 289px;" width="778"> 
 <tbody> 
  <tr> 
   <td width="623"> <p style="padding-left: 30px;"><strong>Highlights</strong></p> <p style="padding-left: 30px;">By default, application-level logs on Outposts rack are not automatically sent to <a href="https://aws.amazon.com/cloudwatch/">Amazon CloudWatch Logs</a> in the Region. You can configure CloudWatch logs agent on Outposts rack to collect and send your application-level logs to CloudWatch logs.</p> <p style="padding-left: 30px;">logs: PutLogEvents does transmit data to the Region, but it is not blocked by the provided SCPs, as it’s expected that most use cases will still want to be able to use this logging API. However, if blocking is desired, then add the action to the first recommended guardrail. If you want specific roles to be allowed, then combine with the ArnNotLike condition example referenced in the previous highlight.</p> </td> 
  </tr> 
 </tbody> 
</table> 
<h2>Conclusion</h2> 
<p>The combined use of Outposts rack and the suggested guardrails via AWS Organizations policies enables you to exercise better control over the movement of the data. By creating a landing zone for your organization, you can apply SCPs to your Outposts racks that will help make sure that your data remains within a specific geographic location, as required by the data residency regulations.</p> 
<p>Note that, while custom guardrails can help you manage data residency on Outposts rack, it’s critical to thoroughly review your policies, procedures, and configurations to make sure that they are compliant with all relevant data residency regulations and requirements. Regularly testing and monitoring your systems can help make sure that your data is protected and your organization stays compliant.</p> 
<h2>References</h2> 
<ul> 
 <li><a href="https://d1.awsstatic.com/whitepapers/compliance/Data_Residency_Whitepaper.pdf">AWS Data Residency Whitepaper</a></li> 
 <li><a href="https://d1.awsstatic.com/product-marketing/Outposts/AWS%20Outposts%20Data%20Residency%20eBook.pdf">AWS Outposts Data Residency eBook</a></li> 
 <li><a href="https://www.youtube.com/watch?v=afPNZThKPwI">Architecting for Data Residency on AWS Outposts Tech Talk</a></li> 
</ul>
