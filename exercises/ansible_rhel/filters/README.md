# Exercise - Working with Ansible Filters


Ansible comes with many filters which allow you to transform data.

Let's go through a couple of common use case examples to showcase their power.


## Parsing JSON Input

Perhaps we've got another application which spits out some valid JSON formatted data, which we want to make use of inside some playbooks for automation.

Let's create some dummy JSON data that we'll use in this example. Run this:

```bash
cat >sample.json << EOF
{
  "apple_pie": "ice_cream",
  "rhubarb": "custard",
  "salt": "pepper",
  "_NULL_": ""
}
EOF
```

Now we have some data to work with, and it should look like this:

```bash
{
  "apple_pie": "ice_cream",
  "rhubarb": "custard",
  "salt": "pepper",
  "_NULL_": ""
}
```


## Setting Defaults and Sanity Checks


---

[Click Here to return to the Ansible Linklight - Ansible Engine Workshop](../README.md)
