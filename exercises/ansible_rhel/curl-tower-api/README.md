# Exercise - Call the Tower API - curl example

## Calling the API to launch a job

We can call into Ansible Tower via the API to run a job template for us.

We'll use the "Install Apache" job template that we created earlier.

### Find the Job ID

We need to find the job ID so we can launch it via the api. In the Tower UI go to **Templates** and then click on **Install Apache** job template. In the url you will see the job ID. For example - https://X.X.X.X/#/templates/job_template/8 - This shows my job ID is **8**.

### Launch the job

We'll use the *curl* command to launch the job. It's a bit of a handful but let's break it down, so it's easier to understand:

```bash
-- user:    who we authenticate as. In our case the admin account
-k:         insecure HTTPS. So we don't check for valid certs
-s:         silent mode. Cuts out some of the not so -useful curl output we don't want
-H:         HTTP JSON MIME type headers. We need to POST in the extra_vars and job_tags so the job will run successfully
```

NB. You will need to check and change where necessary the PUBLIC_IP for your Tower instance and the Job Template number (mine here is 8)

Lastly, we use python to prettify the output, making it more readable.

**NOTE**
Make sure you update the password below from PASSWORD to your Tower admin password. You also need to update the IP address in the URL to be the public IP address of your Tower server. Finally, you also need to update your job template ID. In the below example we are using job template ID 8 - https://X.X.X.X/api/v2/job_templates/`8`/launch/

---


```bash
curl --user 'admin':'PASSWORD' -k -s -H 'Content-Type: application/json' -k -s -XPOST https://X.X.X.X/api/v2/job_templates/8/launch/ | python -m json.tool
```

You should see some output from the job launch including the ID of this particular job run.

```bash
    },
    "timeout": 0,
    "type": "job",
    "unified_job_template": 8,
    "url": "/api/v2/jobs/`58`/",
    "use_fact_cache": false,
    "vault_credential": null,
    "verbosity": 0
}
```

## Checking The Job

From the above output, I can see that **job 58** was created and we can see the **url** to view the status.

```bash
curl --user 'admin':'PASSWORD' -k -s -H 'Content-Type: application/json' -k -s -XGET https://X.X.X.X/api/v2/jobs/58/ | python -m json.tool
```

A lot of output will be returned. This includes the status of the job:

```bash
    "status": "successful",
    "failed": false,
    "started": "2021-07-12T12:42:36.635224Z",
    "finished": "2021-07-12T12:42:51.082975Z",
```

## Summary

You can see from this example that it is very easy to consume the Tower API using basic curl commands. The limitation of this approach is that you need to effectively write your own script to perform the interaction with the Tower API and then parse the output which it returns. You would also need to code things like polling for job completion. We'll look at other ways of consuming the Tower API next which will simplify this process.

---

[Click Here to return to the Ansible Tower Workshop](../README.md)
