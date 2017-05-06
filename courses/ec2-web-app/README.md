# EC2-based Web Application Dev and Production Environment

You have been hired to build a dev and production environment for a company's web application that has a reasonable level of automation, security and resilience.

Building a VPC, subnets, security groups and some ec2 instances a relatively easy, but to be production ready there is a lot more thought and work that needs to go into an environment

## Goal
* By completing the lab you will get to apply a broad range of AWS services that you may not have used in anger previously and consider how best to apply these services in a production environment
* The lab will guide you through the process to standup a complete dev and production environment for a simple web application
  * Automated CI/CD pipeline
  * Security controls in place
  * Resilience to loss of an availability zone but not an entire region. ie All infrastructure for this lab will be within one region.
  * Monitoring and logging

## Pre-requisites
* You have an AWS master account
* You are familiar with AWS concepts and common tools as these labs assume you have some knowledge and hands-on experience
  * Ideally you have completed your Solution Architect Associate Certification
* The lab may use some recently released AWS features which may not be available in all regions. Ensure you select a region that supports:
  * CodeStar
  * CodeCommit
  * CodeBuild
  * CodePipeline
  * Lambda
  * ConfigRules
* This course was developed on Ohio (us-east-2)

