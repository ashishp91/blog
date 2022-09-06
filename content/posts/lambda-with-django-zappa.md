---
title: "Lambda With Django Zappa"
date: 2022-09-04T22:08:13+05:30
tags: ["DevOps", "Lambda","API Gateway","S3"]
draft: false
---

Today we're going to learn how to deploy AWS lambda with Django. Lambda is a event driven, serverless computing platform provided by AWS. Serverless architecture allows us to deploy apps without having to manage the servers.

We're going to use zappa to deploy a Django app to AWS lambda. AWS uses S3 for storing the build and deployment artifacts. zappa will also create an API gateway to provide HTTP endpoints through which the lambda function can be called.

### Setup Django

Make sure you've python3 installed on your system. I experienced issues with python versions other than 3.9. So make sure you're using python 3.9 version.

```
> python3 --version
Python 3.9.13
```

Next we'll setup pip and packages needed to set up AWS lambda.

1. Install pip.
```
python3 -m pip install –upgrade pip
```
2. Install virtualenv. Setup a virtualenv ".env".
```
pip install virtualenv
mkdir lambda-django
cd lambda-django
virtualenv .env
```
3. Switch to the env virtualenv and install packages related to zappa.
```
source .env/bin/activate
pip install awscli
pip install django zappa
```

### Configure AWS

Configure AWS credentials so zappa can access your AWS account.

```
> aws configure
AWS Access Key ID [****************]: ****************
AWS Secret Access Key [****************]: ****************
Default region name [ap-south-1]:
Default output format [json]:
```

Make sure the account you're giving access to have privileges for API Gateway, S3 and Lambda.

### Create django project

Now that we've Django and Zappa installed.

```
django-admin startproject zappatest .
```

Test if everything works by running the Django server.

```
python3 manage.py runserver
```

### Initialize and deploy zappa

Use `zappa init` command inside the folder to initialize zappa settings.

```
cd zappatest
zappa init
```

zappa will ask you to configure stages next, pick the default one 'dev'.

```
Your Zappa configuration can support multiple production stages, like 'dev', 'staging', and 'production'.
What do you want to call this environment (default 'dev'):
```

Next it'll ask you for the AWS credentials. Since we configured it before it'll be stored in the default profile so choose default.

```
AWS Lambda and API Gateway are only available in certain regions. Let's check to make sure you have a profile set up in one that will work.
We found the following profiles: default. Which would you like us to use? (default 'default'):
```

Zappa uses S3 bucket to store your deployed application. You can use the default bucket or give it a name:

```
Your Zappa deployments will need to be uploaded to a private S3 bucket.
If you don't have a bucket yet, we'll create one for you too.
What do you want to call your bucket? (default 'zappa-honv26p9p'):
```

Zappa can deploy your lambda function on all possible regions. You'll not need this. Press enter to select default 'No'.

```
You can optionally deploy to all available regions in order to provide fast global service.
If you are using Zappa for the first time, you probably don't want to do this!
Would you like to deploy this application globally? (default 'n') [y/n/(p)rimary]:
```

Finally this will create a json file `zappa_settings.json`.

```
Okay, here's your zappa_settings.js:
{
    "dev": {
        "aws_region": "ap-south-1",
        "django_settings": "zappatest.settings",
        "profile_name": "default",
        "s3_bucket": "zappa-honv26p9p"
    }
}
Does this look okay? (default 'y') [y/n]:
```

### Deployment with Zappa

Now we'll deploy zappa using command `zappa deploy dev`.

```
Calling deploy for stage dev..
Creating zappatest-dev-ZappaLambdaExecutionRole IAM Role..
Creating zappa-permissions policy on zappatest-dev-ZappaLambdaExecutionRole IAM Role.
Downloading and installing dependencies..
Packaging project as zip..
Uploading zappatest-dev-1496245095.zip (11.0MiB)..
100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 11.5M/11.5M [00:15<00:00, 751KB/s]
Scheduling..
Scheduled zappatest-dev-zappa-keep-warm-handler.keep_warm_callback!
Uploading zappatest-dev-template-1496245132.json (1.6KiB)..
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1.61K/1.61K [00:01<00:00, 1.23KB/s]
Waiting for stack zappatest-dev to create (this can take a bit)..
 75%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▎                                      | 3/4 [00:10<00:05,  5.48s/res]
Deploying API Gateway..
Deployment complete!: https://1kfcd79nh5.execute-api.us-west-2.amazonaws.com/dev
```

You'll get an error when visiting the url returned by zappa deployment "https://1kfcd79nh5.execute-api.us-west-2.amazonaws.com/dev".

```
"{u'message': u'An uncaught exception happened while servicing this request. You can investigate this with the `zappa tail` command.', u'traceback': ['Traceback (most recent call last):\\n', '  File \"/var/task/handler.py\", line 427, in handler\\n    response = Response.from_app(self.wsgi_app, environ)\\n', '  File \"/private/var/folders/ng/z5x6b_t94qg13yx2gh3rzd_h0000gn/T/pip-build-XVZTup/Werkzeug/werkzeug/wrappers.py\", line 903, in from_app\\n', '  File \"/private/var/folders/ng/z5x6b_t94qg13yx2gh3rzd_h0000gn/T/pip-build-XVZTup/Werkzeug/werkzeug/test.py\", line 884, in run_wsgi_app\\n', \"TypeError: 'NoneType' object is not callable\\n\"]}"
```

Add the returned url to `ALLOWED_HOSTS` in settings.py. Now update the deployment by running:

```
zappa update dev
```

Congratulations. You've deployed a Django app to a serverless architecture.
