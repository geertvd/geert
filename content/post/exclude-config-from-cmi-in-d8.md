---
author: "Geert van Dort"
date: 2017-05-02
linktitle: Exclude config from configuration management in Drupal 8
title: Exclude config from configuration management in Drupal 8
weight: 10
tags: ["Drupal", "d8", "CMI"]
---

Drupal 8 makes it really easy to export and import your entire configuration, 
which makes it easily deployable on all your environments. 
However, this is not always desirable when a customer controls part of that configuration.

Let's take the [webform](https://www.drupal.org/project/webform) module as an example.
Webforms are configuration entities in drupal 8, this means that they are automatically
exported when doing a `config-export`.
It also means that webforms which have not been exported will be removed during `config-import`.
This is what we want to avoid.

### Exclude configuration during import
Enter the [config ignore](https://www.drupal.org/project/config_ignore) module.
Config ignore has one configuration page which allows you to enter a number of config entities which should be ignored.
For our webform example we would like to ignore all webform entities, except for the contact webform.  
Our config ignore configuration would look like this:

```
webform.webform.*
~webform.webform.contact
```

Doing this already prevents the configuration from being overridden/removed during import (except for the contact webform).
Just make sure to use the 2.x version of `config_ignore` since 1.x didn't actually prevent the config from being deleted.

### Exclude configuration during export
While `config_import` does a great job at avoiding configuration from being
imported, it doesn't actually prevent that configuration from being exported.
While there's nothing really wrong with having that configuration exported, 
I find it confusing and I'd prefer the ignored configuration to stay out of my repository.

To solve this we can use the [config split](https://www.drupal.org/project/config_split) module.
While `config_split` allows you to export subsets of configuration to be deployed on different environments.
It also allows you to prevent a subset of configuration from being exported at all.
Let's see how we can solve this for our webform example.

I created a new config_split entity, let's call it "Ignored config", I leave the folder blank,
this actually causes the configuration to be exported in the database thus preventing
ignored configuration entities from entering our repository during export.
Besides that I just added the following in the config_split entity's graylist:

```
webform.webform.*
```

Adding it to the graylist rather then the blacklist still allows me to forcefully export some
webform configuration into our repository. (Adding it to the blacklist would remove the configuration
again during the next `config-export`).

Leaving the config_split folder blank is a feature that's only recently ([1d95a8f](https://www.drupal.org/commitlog/commit/88947/1d95a8fc741b8694a42e1e52b0a14e02d67da1f6)) been included,
so at the time of writing I'm using the latest dev release to make use of it.

