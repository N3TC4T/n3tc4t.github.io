---
layout: post
title: Simple Database High Availability for your Django app
categories: posts
comments: true
en: true
description: Simple Database High Availability for your Django app
keywords: "django, postgresql, dbha, availability, failover"
---

# Introduction

__what is HA (high availability) at all ?__


what happen if one of your servers,services,dbs get down , and you slept in bed and you think everything is ok ?

the simplest definition of HA is :

HA make your server stay always up , if one server or service getting down , its will replace it with a alternative healthy server or service

__what about servers replicates?__

if you wanna make your HA system work you should make sure all your db backends have the same level of information, that's mean copying data from a database in one server to a database in another.
so all servers have same data and records

many dbms backend's such as Mysql , SQL Server and Postgresql have their built in replication systems .

__requirements__

* 3 or more replicated database backend server
* 1 django app

# Getting start

first we should define our main dbms and then or alternative or backup dbsm servers to django,
so just need to do something like this in our django settings

_in this example i used postgresql as my dbms backend_


``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'my_database',
        'USER': 'postgre_user',
        'PASSWORD': 'blahblah',
        'HOST': '192.168.1.2',
        'PORT': '5432',
    },
    'failover1': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'my_database',
        'USER': 'postgre_user',
        'PASSWORD': 'blahblah',
        'HOST': '192.168.1.3',
        'PORT': '5432',
    },
    'failover2': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'my_database',
        'USER': 'postgre_user',
        'PASSWORD': 'blahblah',
        'HOST': '192.168.1.4',
        'PORT': '5432',
    }
}
```

with this setting our app just use the default backend (192.168.1.2) for its read and write and if this server get down by any reason our app will stop working

so lets code :)

# Simple code for check our servers status

I just wrote a simple code to check our dbms server status and save current avalaible server in a file.

for reducing requests , checking servers status happen in every 10 second

_file:dbha.py_

``` python
# coding=utf-8
from django.conf import settings
from datetime import datetime
import socket, os


def test_connection_to_db(database_name):
    try:
        db_definition = getattr(settings, 'DATABASES')[database_name]
        s = socket.create_connection((db_definition['HOST'], db_definition['PORT']), 5)
        s.close()
        return True
    except Exception as e:
        return False


def available_db():
    with open(os.path.dirname(__file__) + '/dbha_last_check.txt', 'r') as f:
        f = f.read().splitlines()
        dbha_last_check = datetime.strptime(f[0], '%Y-%m-%d %H:%M:%S')
        last_db = f[1]
    if (datetime.now() - dbha_last_check).seconds > 10:
        if test_connection_to_db('default'):
            db = 'default'
        elif test_connection_to_db('failover1'):
            db = 'failover1'
        elif test_connection_to_db('failover2'):
            db = 'failover2'

        with open(os.path.dirname(__file__) + '/dbha_last_check.txt', 'w') as status_file:
            lines_of_text = datetime.now().strftime('%Y-%m-%d %H:%M:%S') + "\n" + db
            status_file.write(lines_of_text)
            status_file.close()

        return db

    return last_db
```

# creating our custom django database route

django framework have a database route system feature,  so we can use  to handle our fail over and HA system .

first we need to make router.py file in our app root directory with this content

_file:router.py_

``` python
# -*- coding: UTF-8 -*-
import dbha

class ModelDatabaseRouter(object):

    def db_for_read(self, model, **hints):
        """Point all read operations to our available dbms server"""
        return dbha.available_db()

    def db_for_write(self, model, **hints):
        """Point all write operations to our available dbms server"""
        return dbha.available_db()
```

with this code we will get last available dbms server

and in the end we need to tell django to use our router instance it's default router
so just need to put this in our django settings

``` python
DATABASE_ROUTERS = ['app.router.ModelDatabaseRouter']
```


# anything else ?

for sure you can done something like this system on your database side and not application side , you can take a look on [this](/posts/postgresql-replication-repmgr-pgbouncer/)  post for more information to how manage your failover on database side with Repmgr for Postgresql DBMS.