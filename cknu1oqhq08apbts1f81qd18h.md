## Top 6 questions people ask about Django Apps in a Cloud

You have probably tried to look for something regarding Django and Cloud on the Internet — it’s a pretty common way to find an answer to a question. But have you found the right answer to your question? How many search requests did you need to send before you got the answer? How many links did you have to open?


Today, we will take a look at some of the most frequent questions.


For better readability, these questions are not sorted by popularity, but rather by logic.



## Where can I host Django for free?


You can use many cloud solutions for that. But you need to keep in mind that all free solutions have many limitations.


This is a list of some of them:



* CPU resources allocation
* File storage size
* Using custom domain name
* Database limitations


It may become hard to maintain and adjust your use of free solutions to the above constraints and limitations if you choose to deploy your production application using free products.


But free solutions are a good place for prototyping and testing your app.


Here are some of the Cloud solutions:

* Pythonanywhere
* Heroku
* AWS
* Google Cloud Platform



## Which Cloud solution is the best for Django?
From my personal point-of-view, it is kind of a philosophical question.


It depends on the context. First, you need to answer these questions to get an answer to your main question:


* What is your budget?
* What team will work on it (will you engage DevOps or CloudOps Engineers)?
* What database will you use?
*How many users will there be?
*How many requests do you expect to have?


Then, being guided by the answers to those questions, you can try to figure out which solution fits your requirements better.


### For example:
**Case 1.** There are no DevOps or CloudOps Engineers that could help with deploying your app in your team. In this case I’d recommend looking at Heroku.


**Case 2.** There is an AWS CloudOps Engineer in your team, your app should be scalable, you have a good budget for your App: AWS would serve as a good Cloud solution for your Django app.


**Case 3.** You are a student and you just want to try Django. Pythonanywhere allows you to deploy a simple Django app for free. Or you can use Heroku as well.


**Case 4.** You have all your existing infrastructure set up on Google Cloud Platform: it would probably be better to deploy your Django app on that infrastructure.



## How to deploy a Django application to Pythonanywhere?
When I tried doing it for the first time, I used a tutorial from [Django Girls](https://tutorial.djangogirls.org/en/deploy/). But there are many articles about it on the Internet, i.e.:


For example:
* [Pythonanywhere help portal](https://help.pythonanywhere.com/pages/DeployExistingDjangoProject/)
* [Dev.to](https://dev.to/mh_shifat/host-your-django-project-in-pythonanywhere-free-3m4f)
* [Medium](https://medium.com/@AmaanPengoo/uploading-django-to-pythonanywhere-bf4990ae47cc)



## How to deploy a Django application to Heroku?
[Heroku Dev Center](https://devcenter.heroku.com/articles/deploying-python) already has a good answer. Also you can find many answers to this question on Dev.to, medium.com, and many other resources.



## How to deploy a Django application to Google Cloud?
It is possible to run Django apps on  [App Engine standard environment](https://cloud.google.com/appengine)  scale dynamically according to the traffic. Here is [a good tutorial from Google about how to do it](https://cloud.google.com/python/django/appengine))



## How to deploy a Django application on AWS?
From my personal experience, it is the hardest task. There are a couple of AWS services where you can deploy your Django App. You can use Amazon Elastic Compute Cloud (Amazon EC2) and AWS Lambda.


And there is more than one way to deploy it there.


It will be laborious to explain even one of the methods within the scope of these paragraphs, but in my following articles I will make an effort to address the points below step by step:

* prepare Django app
* set up all the necessary AWS resources
* configure AWS to serve static files
* and others…



## Summary
There are many solutions regarding deploying a Django App in a Cloud. You can easily use one of them that fits better to your requirements. 

I'd like to share some pieces of advice for looking for the right answer:

* be specific enough in your question
* compare your requirements with the suggested requirements (python or Django versions)
* review comments and likes to check what other developers think about the answers provided
* look at author of the answer (check whether they have enough expertise to answer that question).


In my following blog posts, I will describe how to deploy Django on AWS lambda using Serverless. Follow me not to miss that. 

Good luck!