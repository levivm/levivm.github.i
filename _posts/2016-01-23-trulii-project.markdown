---
title: "Trulii"
layout: post
date: 2016-01-23 22:10
tag: projects
img: indigo/indigo.png
projects: true
description: "A marketplace where you can find hundreds of classes in your city"
jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

<center> ![Screenshot](https://s32.postimg.org/keod9jqmd/Home_Desktop.jpg)


Trulii as platform is split in two main sections. We built backend as an API REST using Django-Rest-Framework in order to be consumed by any mobile or web app. The client side is a SPA built using AngularJS. 

Both parts are totally containerized using docker/docker-compose and hosted on AWS EC2 instances. Integrating docker was a big challange but now we can deploy a new server in just five minutes.

Frontend was built following strictly [John Papa's Angular Style Guide](https://github.com/johnpapa/angular-styleguide) to guarantee a clean, correct and readable code. 


---

What has inside?

Server and Deployment:

- Docker
- AWS
- Nginx

Backend:

- Django/Python
- Django-Rest-Framework
- Redis
- Celery
- Gunicorn
- PostgreSQL

Frontend:

- Angular
- Gulp
- Less
- Mongo DB
- NodeJS

---

[Check it out](trulii.com) here.
If you want to know a bit more, just [tell me](mailto:levinoelvm@gmail.com).