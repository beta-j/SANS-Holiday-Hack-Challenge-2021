# OBJECTIVE 10 - Now Hiring! #

## OBJECTIVE : ##
>What is the secret access key for the [Jack Frost Tower job applications server](https://apply.jackfrosttower.com/)? Brave the perils of Jack's bathroom to get hints from Noxious O. D'or. 
#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 10</summary>
  
>-  The [AWS documentation for IMDS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) is interesting reading.
</details>

#  

## PROCEDURE : ##

We start off with a very helpful hint in what Noxious O D’Or tells us before giving us the official hints for this objective; *“Dr. Petabyte told us, ‘Anytime you see URL as an input, test for SSRF.’”*.

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/8a5442bb-084e-4d48-aa4b-109ffcbfe499)

This is particularly helpful since the Jack Frost Tower application form includes a field which expects a URL to a public *“NLBI Report”* as input

Right away I notice that by placing some typical SSRF strings[^1] in this field I get a different output page on submission and that the page is trying to display an image called `<name>.jpg` (where `name` is the name entered in the application form).

In particular it looks like strings such as `file:///etc/passwd` and `http://169.254.169.254/latest/meta-data/` trigger this kind of behaviour so I know for sure that the website is vulnerable to SSRF and is most likely using AWS.  In order to retrieve the results of these queries, I run the web form through Burp Suite so that I can see the raw data within `<name>.jpg`.

This way I was able to retrieve the contents of `/etc/passwd` and `http://169.254.169.254/latest/meta-data` as expected.
Similarly by submitting `http://169.254.169.254/latest/meta-data/security-credentials` I found a user called `jf-deploy-role` and presumably `jf` stands for our old enemy; **Jack Frost**.  So by going back to the application form and submitting `http://169.254.169.254/latest/meta-data/security-credentials/jf-deploy-role` I got the following raw output:
```
	"Code": "Success",
	"LastUpdated": "2021-05-02T18:50:40Z",
	"Type": "AWS-HMAC",
	"AccessKeyId": "AKIA5HMBSK1SYXYTOXX6",
	"SecretAccessKey": "CGgQcSdERePvGgr058r3PObPq3+0CfraKcsLREpX",
	"Token": "NR9Sz/7fzxwIgv7URgHRAckJK0JKbXoNBcy032XeVPqP8/tWiR/KVSdK8FTPfZWbxQ==",
	"Expiration": "2026-05-02T18:50:40Z" 
```

And there it is – Jack Frost’s `SecretAccessKey` in all its glory.  Super easy when you’ve completed the IMDS Exploration Challenge 😊

`"SecretAccessKey":` **`"CGgQcSdERePvGgr058r3PObPq3+0CfraKcsLREpX",`**

[^1]:This webpage was particularly useful for this bit: [https://cobalt.io/blog/a-pentesters-guide-to-server-side-request-forgery-ssrf](https://cobalt.io/blog/a-pentesters-guide-to-server-side-request-forgery-ssrf ) 
