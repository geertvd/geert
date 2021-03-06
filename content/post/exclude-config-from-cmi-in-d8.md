---
author: "Geert van Dort"
date: 2017-05-03 22:00:00
linktitle: Exclude config from configuration management in Drupal 8
title: Exclude config from configuration management in Drupal 8
weight: 10
tags: ["Drupal"]
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
Config ignore has one configuration page which allows you to enter a number of config entities that should be ignored.
For our webform example we would like to ignore all webform entities, except for the contact webform.  
Our config ignore configuration would look like this:

```
webform.webform.*
~webform.webform.contact
```

Doing this already prevents the configuration from being overridden/removed during import (except for the contact webform).
Just make sure to use the 2.x version of config ignore since 1.x didn't actually prevent the config from being deleted.

### Exclude configuration during export
While config ignore does a great job at avoiding configuration from being
imported, it doesn't actually prevent that configuration from being exported.
While there's nothing really wrong with having that configuration exported, 
I find it confusing and I'd prefer the ignored configuration to stay out of my repository.

To solve this we can use the [config split](https://www.drupal.org/project/config_split) module.
While config split allows you to export subsets of configuration to be deployed on different environments.
It also allows you to prevent a subset of configuration from being exported to files.
Let's see how we can solve this for our webform example.

I created a new config split entity, let's call it "Ignored config", I leave the folder blank,
this actually causes the configuration to be exported in the database thus preventing
ignored configuration entities from entering our repository during export.  
I also set a negative weight (I set mine to -10), we have to do this to prevent our ignored config split entity
from overruling the configuration ignored by the config_ignore module. 
(Thanks to [albertski](https://www.drupal.org/u/albertski) for noticing this [issue](https://www.drupal.org/node/2883110)).  
Besides that I just added the following in the config split entity's graylist:

```
webform.webform.*
```

Adding it to the graylist rather then the blacklist still allows me to forcefully export some
webform configuration into our repository. (Adding it to the blacklist would remove the configuration
again during the next `config-export`).

Leaving the config split folder blank is a feature that's only recently ([1d95a8f](https://www.drupal.org/commitlog/commit/88947/1d95a8fc741b8694a42e1e52b0a14e02d67da1f6)) been included,
so at the time of writing I'm using the latest dev release to make use of it.

