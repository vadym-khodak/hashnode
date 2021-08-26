## Deploying a Django project on AWS Lambda using Serverless (Part 4)

As I promised in my previous blog post [Deploying a Django project on AWS Lambda using Serverless (Part 3)](https://blog.vadymkhodak.com/deploying-a-django-project-on-aws-lambda-using-serverless-part-3), I'd like to show you how to add a React.JS client to a Django project and deploy it with Django on AWS Lambda using Serverless.

##  [BLUF](https://bit.ly/38hbFUC)
Django framework allows you to build a client using Django templates, but there are a lot of cases when this is not enough. Business requirements for the client-side of your application could require adding more complex logic on the client-side. In these cases, we will not be able to solve business problems without a JavaScript web framework (React.JS, Vue.JS, Angular, etc). I'd like to show you how to build a simple React.JS client and integrate it with a Django project using [`axios`](https://www.npmjs.com/package/axios) library on the client-side and [Django REST Framework](https://pypi.org/project/djangorestframework/) on the server-side.

With this approach, I will build a React.JS client with a domain name of CloudFront distribution as a PUBLIC_URL and store it on AWS S3 bucket together with Django static files. Then, I add the built `index.html` file with React.JS app to Django templates folder and deploy it with the Django project on AWS Lambda. 

## Getting started

I've already described how to create a Django project and deploy it on AWS Lambda using Serverless in [my first blog post](https://blog.vadymkhodak.com/deploy-django-app-on-aws-lambda-using-serverless-part-1) of this series. I will use that project to get started. 

Let's go through this process step by step:

* Clone the Django repository I used in [the first part of these series](https://github.com/vadym-khodak/django-aws-lambda) and go to the cloned repository:

```bash
git clone https://github.com/vadym-khodak/django-aws-lambda
cd django-aws-lambda
```

* Follow instructions from [this blog post](https://blog.vadymkhodak.com/deploy-django-app-on-aws-lambda-using-serverless-part-1) to run the Django server (install requirements, configure environment variables using `.env`, apply migrations, create a superuser, collect static, run the server).

If everything works, you are ready to start working on the client.

## Add React.JS client 
In this part, I'll show how to create a simple React.JS client and integrate it with our Django project, but I'm sure that you can easily use Vue.JS (I'm not familiar with Angular) as steps pretty much the same.
 
There are many options to create React.JS client. I'm using [`react-scripts`](https://www.npmjs.com/package/react-scripts) in this example.

* Install [`react-scripts`](https://www.npmjs.com/package/react-scripts) using [`npm`](https://www.npmjs.com/) (Node package manager)

```bash
npm install -g react-scripts
```

* Use react-scripts to initialize a React.JS client

```bash
npm init react-app client
```

* Check that React.JS client was built successfully

```bash
cd client
npm run start
```

It will open your browser on `localhost` port `3000` and you will see a page like this:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628417085242/Wn168Hb1y.png)

## Update Django project configuration
Let's update our Django configuration to render `index.html` file with React.JS client:

* Add  `CLIENT_DIR` environment variable to `.env` file:

```
CLIENT_DIR="client/build"
```

* Update `django_aws_lambda/urls.py` file with the following code:

```python
from django.contrib import admin
from django.urls import path, include
from django.views.generic import TemplateView


urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/', include('hello.urls')),
    path('', TemplateView.as_view(template_name="index.html"), {'resource': ''}),
    path('<path:resource>', TemplateView.as_view(template_name="index.html")),
]
```

* Update `STATICFILES_DIRS` in `django_aws_lambda/settings.py`

```python
STATICFILES_DIRS = [
    str(ROOT_DIR / env('CLIENT_DIR', default='client/build')),
    str(ROOT_DIR / 'static'),
]
```

## Check that our Django project can run the React.JS client

* Build production React.JS client locally the following commands:

```bash
cd client
export PUBLIC_URL=static/
npm run build
cd ..
```

* Collect static files by running the following command:

```bash
python manage.py collectstatic --no-input
```

* Run this Django project locally (make sure that you have already applied migrations and created a superuser):

```bash
python manage.py runserver
```

* Go to your browser, open this URL [http://localhost:8000](http://localhost:8000) and you will see a page like this:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628429290188/jESwmYqu2.png)

It looks the same as we saw before by running `npm run start` in `client` folder. There are just a couple of differences - now it is run on port `8000` and it is run by our Django web server.

## Make the React.JS client talk to the Django server.
First, we need to create an API endpoint to return some data from the server to the client. The easiest way to build REST API in Django is using [Django REST framework](https://pypi.org/project/djangorestframework/) project.

* Install Django REST framework and add it to `requirements.txt` file

```bash
pip install djangorestframework
```

* Create a new Django app called `users` by running the following command:

```bash
python manage.py startapp users
```

* Update `users/views.py` file with the following code:


```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def get_current_user(request):
    return Response(status=status.HTTP_200_OK, data={"username": request.user.username})
```

> This is an endpoint that returns Response object with `username` (if the user is not authorized it will return an empty string as `username`)

* Update `users/urls.py` file with the following code:

```python
from django.urls import path

from .views import get_current_user

app_name = "users"
urlpatterns = [
    path("me/", view=get_current_user, name="get-current-user"),
]
```

* Update our Django project configuration

Update `django_aws_lambda/urls.py` file with the following code:

```python
from django.contrib import admin
from django.urls import path, include
from django.views.generic import TemplateView


urlpatterns = [
    path('admin/', admin.site.urls),
    path('hello/', include('hello.urls')),
    path('api/users/', include('users.urls')),
    path('', TemplateView.as_view(template_name="index.html"), {'resource': ''}),
    path('<path:resource>', TemplateView.as_view(template_name="index.html")),
]
```

* Update `INSTALLED_APPS` in `django_aws_lambda/settings.py`

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'hello',
    'rest_framework',
    'users',
]
```

### Modifying React.JS client to send requests to the Django server

* Install `axios` library:

```bash
cd client
npm install axios -S
```
> `-S` flag will add `axios` library to `package.json` file

* Update `client/src/App.js` file with the following code:

```javascript
import { useEffect, useState } from 'react';
import axios from 'axios';
import logo from './logo.svg';
import './App.css';

function App() {
  const loadUserDetails = async () => {
    const response = await axios.get('/api/users/me/');
    return response.data;
  };
  const [userData, setUserData] = useState(false);
  useEffect(() => {
    loadUserDetails().then((payload) => {
      setUserData(payload);
    });
  }, []);

  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
        <h1>Hello, World!</h1>
        <h2>I'm {(userData && userData.username) || 'Unknown User'}</h2>
      </header>
    </div>
  );
}

export default App;
```

* Build production optimized client by running the following command:

```bash
export PUBLIC_URL=static/
npm run build
cd ..
```

* Collect static files by running the following command:

```bash
python manage.py collectstatic --no-input
```

* Run this Django project locally:

```bash
python manage.py runserver
```

Go to your browser, open this URL [http://localhost:8000](http://localhost:8000) and you will see a page like this:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628435119388/Wv7F81VzgE.png)

But if you authorize using your superuser username and password you will see a page like this:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628435553669/m4_GAmqiR.png)

## Deploying on AWS Lambda using Serverless
#### Prepare AWS infrastructure
I've already described how to prepare AWS infrastructure in my previous blog posts, so you can use one of the following approaches: 

* Prepare AWS infrastructure manually as it is described in [Deploying a Django project on AWS Lambda using Serverless (Part 3)](https://blog.vadymkhodak.com/deploying-a-django-project-on-aws-lambda-using-serverless-part-3) blog post
* Prepare AWS infrastructure automatically using terraform as it is described in [Deploying a Django project on AWS Lambda using Serverless (Part 2)](https://blog.vadymkhodak.com/deploy-django-app-on-aws-lambda-using-serverless-part-2) blog post

#### Update Serverless configuration

Add `client` folder to `package.exclude` to exclude it from deployment

#### Update URL in `client/src/App.js` file to be able to send requests to production server

```javascript
    const response = await axios.get('/production/api/users/me/');
```

#### Use Docker for deploying your Django project to AWS Lambda using Serverless

* Prepare Amazon Linux 2 docker image with all the necessary dependencies:

```bash
docker run -it -v $(pwd):/root/src/ -v /Users/<your-username>/.aws:/root/.aws amazonlinux:latest bash
# install the necessary Unix dependencies:
yum install sudo -y
sudo yum install -y gcc openssl-devel bzip2-devel libffi-devel wget tar sqlite-devel gcc-c++ make
# install node.js version 14:
curl -sL https://rpm.nodesource.com/setup_14.x | sudo -E bash - 
sudo yum install -y nodejs
# install Python 3.8.7:
cd /opt
sudo wget https://www.python.org/ftp/python/3.8.7/Python-3.8.7.tgz
sudo tar xzf Python-3.8.7.tgz
cd Python-3.8.7
sudo ./configure --enable-optimizations
sudo make altinstall
sudo rm -f /opt/Python-3.8.7.tgz
# create python and pip aliases:
alias python='python3.8'
alias pip='pip3.8'
# update pip and setuptools:
pip install --upgrade pip setuptools
# install serverless:
npm install -g serverless
# move to project directory
cd /root/src/
# install requirements inside docker container:
pip install -r requirements.txt
# set the necessary environment variables
export DJANGO_SETTINGS_MODULE=django_react_aws_lambda.production
export AWS_ACCESS_KEY_ID=<your-aws-access-key-id>
export AWS_SECRET_ACCESS_KEY=<your-aws-secret-access-key>
# migrate database changes
python manage.py migrate
# create a superuser in the database
python manage.py createsuperuser
# build React.JS client for AWS Lambda
cd client
npm install 
export PUBLIC_URL="https://<your-cloud-front-distribution>.cloudfront.net/static/"
npm run build
# copy `index.html` from `client/build` to `templates`
cp build/index.html ../templates/index.html
cd ..
# collect static files to AWS S3 bucket
python manage.py collectstatic  --no-input
# install serverless packages from package.json
npm install
# deploy your Django project to AWS Lambda using Serverless
serverless deploy -s production
```

Now, your Django Project with React.JS client will be available at this URL: 
`https://<some-id>.execute-api.<your-aws-region>.amazonaws.com/production`

Without authorized user:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628443592093/wlm7XuMpl.png)

With authorized user:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628443889989/Nc23jbKJez.png)

## How it works
When you go to the Django project URL on your browser it will go to AWS API Gateway that will trigger AWS Lambda function with the Django WSGI server. The Django server will render `index.html`  with React.JS app, the browser will use the domain name of the CloudFront distribution to get React.JS client from the AWS S3 bucket.

## Final Words
In this blog post, we saw how to add the React.JS client to the Django project and deploy them on AWS Lambda using Serverless. There is a link to the [GitHub repository](https://github.com/vadym-khodak/django-react-aws-lambda) ([GitLab copy](https://gitlab.com/vadym-khodak/django-react-aws-lambda)) with the code shown in this blog post.

It is the final blog post in this series. I showed just one of many ways to deploy a Django project on AWS Lambda, prepare AWS infrastructure, and add a React.JS client. You can find many other ways to do the same, it is up to you which approach to use.

Don't forget to follow me on Twitter [@vadim_khodak](https://twitter.com/vadim_khodak) or on [LinkedIn](https://www.linkedin.com/in/vadym-khodak-0b1a05149/) so you do not miss the next posts.


