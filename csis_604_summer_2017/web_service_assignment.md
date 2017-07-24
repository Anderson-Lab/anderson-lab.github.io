---
permalink: /csis_604_summer_2017/web_service_assignment/
title: "Web Service Assignment"
excerpt: "Anderson Data Science Research Lab."
---

{% include base_path %}

In this assignment, we will be creating a restful webservice and interacting with it using CURL. For our webservice, we will use RESTful paradigm, which means our app only has GET, POST, PUT, and DELETE operations. There are a lot of different options here. Perfect for us, is developing a RESTful endpoint using Google App Engine Endpoints (<a href="https://cloud.google.com/endpoints/docs/tutorials-app-engine-standard">https://cloud.google.com/endpoints/docs/tutorials-app-engine-standard</a>). I'll let you use either the Java or Python tutorial. To put it simply, I want you to do the following two things:

1. Follow and complete the tutorial linked above for either Java or Python (<a href="https://cloud.google.com/endpoints/docs/tutorials-app-engine-standard">https://cloud.google.com/endpoints/docs/tutorials-app-engine-standard</a>). 
2. Once you've got that experience under your belt, I want you to extend the echo example, so that it serves as a blog backend endpoint. You'll need to add a way to create a new blog entry, delete an entry, edit an entry, and retrieve and entry. Entries are just plain text. You'll need to decide on some backend storage. That as well as a lot of other design decisions, I leave entirely up to you. You do NOT need to build an interface or frontend of any kind. You must use CURL and CURL only to interact with your endpoint. 

Instead, of the usual submission protocol of submitting a write-up, you must create a GitHub repo and upload your code to it. You must edit and include a README.md file in the root of the repo that explains how to access your API, which should be live on Google App Engine and available for me to test with your sample commands. If you are unfamiliar with Git, it's super easy to pick up the basics. I suggest you stick to the command line version as those skills will transfer anywhere. There are frontend programs but who needs those crutches? Here is a video to get you started (<a href="https://www.youtube.com/watch?v=HVsySz-h9r4">https://www.youtube.com/watch?v=HVsySz-h9r4</a>. There are a lot of other quickly available tutorial online if that isn't to your liking.
