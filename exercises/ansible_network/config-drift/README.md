# Exercise 4-5: Configuration Drift

## Table of Contents

- [Objective](#objective)
- [Guide](#guide)
- [Solution](#solution)

# Objective

In this exercise we will use Ansible to check the configuration of our routers. If we detect any drift then we will put the device back into our desired state and log an incident in ServiceNow.

# Guide

## Step 1:

To use the ServiceNow module we need to install the pysnow python module. On your Ansible Control/Tower node install the module.

```bash
sudo pip install pysnow 
```

## Step 2:

First we Open the web UI and click on the `Templates` link on the left menu.

![templates link](images/templates.png)

Click on the green `+` button to create a new job template (make sure to select `Job Template` and not `Workflow Template`)

| Parameter | Value |
|---|---|
| Name  | ROUTER CONFIG  |
|  Job Type |  Run |
|  Inventory |  Workshop Inventory |
|  Project |  Workshop Project |
|  Playbook |  router_config.yml |
|  Credential |  Workshop Credential |

Now add the following into EXTRA VARIABLES:
```bash
snow_user: <instructor to provide>
snow_password: <instructor to provide> 
snow_instance: <instructor to provide>
```

Scroll down and click the green `save` button.


## Step 3

Now let's launch the job template. We shouldn't see any changes being made during the job run and as a result we don't need to raise an incident in ServiceNow.

## Step 4

Let's log onto one of our routers and make a change outside of the control of our automation. 

```bash
ssh rtr1
rtr1#conf t
rtr1(config)#ip ssh time-out 120
```

Now execute our **ROUTER CONFIG** job again. This time we should see some changes being made and a ServiceNow incident should be raised. The incident number is reported as part of the job output as well.

![job_link](images/snow_output.png)

## Step 5

Log into ServiceNow - https://<< instance name >>.service-now.com. Use the same credentials you passed into your job template as variables.

In the left hand pane, click on incidents and you should see your incident number. Click on your incident and you will see a comment in the ticket that says "Configuration drift detected in tower job ID XX". This job ID matches the Job ID in your Tower instance so you can easily tie the ServiceNow incident with the Ansible Tower job.

# Solution
You have finished this exercise. We have used Ansible to check the configuration of our routers and report any configuration drift as an incident in ServiceNow.

[Click here to return to the lab guide](../README.md)
