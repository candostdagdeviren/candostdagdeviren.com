---
layout: single
title: "Managing Multiple AWS Environments with Terraform"
date: 2019-03-25
categories: [Software Development, English]
tags: [AWS, Terraform, Software Development, DevOps, Best Practices]
header:
  overlay_image: assets/images/managing-multiple-aws-environments-with-terraform/cover.jpx
  overlay_filter: rgba(0, 0, 0, 0.44)
  show_overlay_excerpt: false
  caption: Photo by [Fancycrave](https://unsplash.com/photos/iWzC3nQXvAY) on [Unsplash](https://unsplash.com)
excerpt: "Multiple environments like staging, production are standard and when we use it with AWS, we face the problem of managing access to them."
meta: "Multiple environments like staging, production are standard and when we use it with AWS, we face the problem of managing access to them."
read_time: true
share: true
related: true
---

Multiple environments like staging, production, development are standard in the software development community. When we combine this approach with AWS usage, we come across a problem where we have to define a user for each environment. We either create a new IAM user separately under the same organization or use multiple organizations. There are two other approaches which we shouldn't do it at all, using an admin or creating one user for all.

In my opinion, having one organization and using different users for different environments make more sense. In here, we're going to take a look at how we are going to approach this process by using Terraform. Terraform is a tool for managing infrastructure. We can easily create, change and improve the infrastructure by code instead of using the AWS Console. AWS Console is incredible, simple and it's perfect for smaller projects. It has all the features we need. The reason why we use Terraform is we want to see the infrastructure resources in our repository. So, everyone understands which tools we're using from AWS by only looking to the code.

> Using Identity and Access Management (IAM), you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.
While IAM allows us to manage access to resources, Terraform has a Workspaces feature which is designed for our problem. Each workspace can be used as an environment in the project. We can add a new one, switch from one to another quickly. But how we're going to use it? First, let's take a look at how to create an IAM user without multiple environments/workspaces.

## Creating Resources

After creating a new Terraform file named users.tf, we should define the resources (users) which we will use. We define an `iam_user` variable and give a default name and description. And we define an `aws_iam_user` resource for our defined user name.

```
variable "iam_user" {
  default     = "candost"
  description = "My AWS IAM user"
}
```

Now we define a `aws_iam_user` resource for our defined user. In here we use the variable in the name field for our resource.

```
resource "aws_iam_user" "candost" {
  name = "${var.iam_user}"
  path = "/"
}
```

Lastly, we define a `aws_iam_user_policy` to attach necessary resource permissions to our brand new user. We give our user name in user property as `aws_iam_user.candost.name`. This will fetch the name we defined with `user` variable. We assume that we have defined a DynamoDB Table named _MyTable_. I'll explain the setup of that table later.

```
resource "aws_iam_user_policy" "candost_policy" {
  name_prefix = "${var.iam_user}"
  user        = "${aws_iam_user.candost.name}"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "dynamodb:*",
      "Resource": [
        "arn:aws:dynamodb:eu-west-1:113608568902:table/MyTable"
      ]
    }
  ]
}
EOF
}
```

## Terraform Workspaces

Now, let's add the environment. Terraform gives access to current workspace with `${terraform.workspace}` variable. We'll append this to our resources. Here is the final version with multiple environments.

```
variable "iam_user" {
  default     = "candost"
  description = "My AWS IAM user"
}

resource "aws_iam_user" "candost" {
  name = "${var.iam_user}-${terraform.workspace}"
  path = "/"
}

resource "aws_iam_user_policy" "candost_policy" {
  name_prefix = "${var.iam_user}-${terraform.workspace}-"
  user        = "${aws_iam_user.candost.name}"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "dynamodb:*",
      "Resource": [
        "arn:aws:dynamodb:eu-west-1:113608568902:table/MyTable"
      ]
    }
  ]
}
EOF
}
```

As we can see, we changed nothing except combining the user name and the Terraform workspace. But they still access the same table. We need to separate that table too. Let's take a look at how we add a resource for the DynamoDB table. We create a *database.tf* file and put the following code in there.

```
resource "aws_dynamodb_table" "MyTable" {
  name           = "MyTable-${terraform.workspace}"
  read_capacity  = 20
  write_capacity = 20
  hash_key       = "myHashKey"

  attribute {
    name = "myHashKey"
    type = "S"
  }
}
```

Now, we have a table code which will work with multiple environments and it will help us to create different tables for different environments. **So, one code will generate multiple environments setup.** Let's update our policy for the new DynamoDB table resource and finish up. Here is the final version:

```
variable "iam_user" {
  default     = "candost"
  description = "My AWS IAM user"
}

resource "aws_iam_user" "candost" {
  name = "${var.iam_user}-${terraform.workspace}"
  path = "/"
}

resource "aws_iam_user_policy" "candost_policy" {
  name_prefix = "${var.iam_user}-${terraform.workspace}-"
  user        = "${aws_iam_user.candost.name}"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "dynamodb:*",
      "Resource": [
        "arn:aws:dynamodb:eu-west-1:113608568902:table/MyTable-${terraform.workspace}"
      ]
    }
  ]
}
EOF
}
```

Now, we can use the following commands to create a new workspace and apply our changes to AWS.

```bash
terraform workspace new stage
terraform workspace select stage
terraform plan
terraform apply
```

Terraform gives a lot of opportunities to manage infrastructure resources easily. Infrastructure as a code is a neat approach and allows developers to share the code, follow the changes easily and understand the current resources used inside the project.

What do you think about using the Terraform Workspace feature? How do you manage access to resources for multiple environments? Let me know your thoughts, comments or feedback on Twitter [@candosten](https://twitter.com/candosten) or comments below.
