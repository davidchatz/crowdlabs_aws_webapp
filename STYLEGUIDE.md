# Style Guide

This a style guide for content contributed to this project that attempts to prescribe only what is needed to allow contributions from different people to work together. 

## Lab notes
* All lab notes to be written in mark down
* Link to AWS and other documentation where relevant

## Cloudformation
* All cloudformation code to be in YAML

## Naming convention
There is no right or wrong naming convention, the important this is to have one and stick by it.

* Use dash "-" as the separator in resource names
* Use only lower case letters
  * Camel case does not work for S3 buckets which must be lower case 

* Prefix the resource name with what it is, for example
  * ec2 - ec2 instance
  * sg - security group
  * sn - subnet
  * rt - route table
  * vpc - VPC
  * trail - CloudTrail trail

## File structure
* Each course is contained in a directory
* Each lab is contained in its own directory within the course directory
* Use a README.md file in each directory to describe each course/lab
