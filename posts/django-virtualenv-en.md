---
layout: post
title:  "Start Django with Virtualenv"
subtitle: ""
author: "Siner"
header-img: "img/django/django-virtualenv.jpg"
catalog: true
header-mask:  0.3
tags:
    - python
date:   2019-01-12
multilingual: false
---

This post is based on **Ubuntu 16.04**.

---

Here's what you need to configure a **Django** project using **virtualenv**.<br>
- **pip**
- **virtualenv** (**Python** >= 3.5)
- **Django** >= 2.0

---

## 0) Why virtualenv?
There are people who say that **"Django works fine, Even if you are not using virtual environments"**

Let's assume the situation.

**Without a virtual environment**, you're installing and using pip modules.<br>
At that time, you installed the latest version of **Django 1.11**, using a new feature called **A**.<br>
A few years later, as **Django 2.2** becomes available and new features are added, the previously used feature **A** is no longer available.<br>
You want to use **Django 2.2** in a new project, so you upgrade Django 1.1, which was previously installed **without a virtual environment, to 2.2**.<br>
At that very moment, the **A** feature you were using in a project that was being deployed in Django 1.11 would not work.

If you use a virtual environment, you can use **multiple versions** of python on the **same host** at the same time,<br>
Therefore, you can use **multiple versions** of Django at the same time.<br>

![python_virtualenv](https://user-images.githubusercontent.com/34048253/51121149-0a461880-185a-11e9-8a5d-6ec7ed58aa60.png)

Now, let's start a Django project with virtualenv!

---

## 1) Install pip
Follow the command below to install pip.

```python
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py
```

![image](https://user-images.githubusercontent.com/34048253/51082217-18a31000-1746-11e9-94b3-2cdfb6039ca4.png)

---

## 2) Install virtualenv
If you have pip installed, this time you will install **virtualenv** using pip.

```python
pip install virtualenv
```

![image](https://user-images.githubusercontent.com/34048253/51082225-3a9c9280-1746-11e9-9bc4-91a05049b0de.png)

---

## 3) Create virtualenv
```python
virtualenv -p python3 venv_for_django
```

If you do not use the -p option, it is recommended to specify the version, since virtualenv can be created based on python2. 

![image](https://user-images.githubusercontent.com/34048253/51082232-4be59f00-1746-11e9-8eac-5f11b63184f6.png)

---

## 4) Activate virtualenv
```python
source venv_for_django/bin/activate
```

![image](https://user-images.githubusercontent.com/34048253/51082234-59028e00-1746-11e9-8200-7bce0f604007.png)

---

## 5) Check python version
Make sure that the virtualenv we created is based on python3 as intended.

```python
python --version
```

To use **Django** 2.0 or later, you must have a version of **python3** installed in virtualenv.<br>
So, when you create virtualenv, put the **-p python3** option.

If you do not include this option and you install a version of python2, **a lower version of Django (1.11) will be installed.**

![image](https://user-images.githubusercontent.com/34048253/51787983-a3125780-21bb-11e9-9898-f5a64d383b84.png)
![image](https://user-images.githubusercontent.com/34048253/51082240-89e2c300-1746-11e9-9444-018887dd64b2.png)

---

## 6) Install Django
Install Django using pip.
```python
pip install django
```
![image](https://user-images.githubusercontent.com/34048253/51082246-9c5cfc80-1746-11e9-856e-11fbf902c6a8.png)

---

## 7) Create Django project
In the virtualenv environment whrer Django is installed, enter the following command.
```python
django-admin startproject myproject
```

If the project name appears to be invalid as shown in the screenshot below, try a different project name.

![image](https://user-images.githubusercontent.com/34048253/51082256-d0382200-1746-11e9-86bc-5dd2ca6659bc.png)

---

## 8) Start Django project
Let's go into the generated project and run **python manage.py runserver** with **ip** and **port** options in the basic command.<br>
(I entered ip with 0.0.0.0 for external connection and entered port number 8000.)
```python
python manage.py runserver 0.0.0.0:8000
```
You will see a warning about **migrate** as shown in the screenshot below, but you can ignore it right now.<br>
We'll talk about **makemigrations** and **migrate** later.

![image](https://user-images.githubusercontent.com/34048253/51082269-01b0ed80-1747-11e9-84fe-e045bdb44691.png)

---

## 9) ALLOWED_HOSTS
If you connect to host ip through port 8000 through web browser, you can see the debug screen with error **DisallowedHost at /**.

![image](https://user-images.githubusercontent.com/34048253/51082279-5d7b7680-1747-11e9-9674-6e49b2eb9612.png)

Since we do not have an ip on the **ALLOWED_HOSTS** list, we need to add 192.168.0.3, the ip we will connect to.<br>
The ALLOWED_HOSTS setting exists in **myproject/settings.py**.

![image](https://user-images.githubusercontent.com/34048253/51082397-76852700-1749-11e9-9d75-9a4853fee34d.png)

Enter **'*'** in **ALLOWED_HOSTS** to allow access even if the host's address (ip, domain) changes.

![image](https://user-images.githubusercontent.com/34048253/51082295-be0ab380-1747-11e9-87cb-fca9b5e81e32.png)

---

## 10) Result
After the modification, run the **runserver** command again and you will see that the **Django** server is running normally.
![image](https://user-images.githubusercontent.com/34048253/51082317-0f1aa780-1748-11e9-91b7-2a6c99a98b4b.png)
