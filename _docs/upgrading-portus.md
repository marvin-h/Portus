---
title: Upgrading Portus
layout: post
order: 6
longtitle: How to upgrade Portus across different versions
---

## Upgrading from 2.0 to 2.1

### Changes on the configuration

First of all, one of the breaking changes from 2.0 to 2.1 is that the machine
FQDN is no longer considered a secret, but a mere configuration value. In order
to provide a smoother transition, in Portus 2.0.5 we allow the FQDN to be
specified in either the `config/secrets.yml` file or as a proper configurable
value (see [this](https://github.com/SUSE/Portus/commit/f0850459cc43e9b9258e70867d5608f2ef303f3e) commit).
This configurable option is defined like this:

{% highlight yaml %}
machine_fqdn:
  value: "portus.test.lan"
{% endhighlight %}

Therefore, just provide the same value as you had from the `config/secrets.yml`
file, into the `config/config-local.yml` file or the `PORTUS_MACHINE_FQDN_VALUE`
environment variable.

Another breaking change is the `jwt_expiration_time` configuration option. Now,
instead of being a standalone option, it's been put inside of the `registry`
configuration. So, now instead of:

{% highlight yaml %}
jwt_expiration_time
  value: "5.minutes"
{% endhighlight %}

The current configuration is like this:

{% highlight yaml %}
registry:
  jwt_expiration_time:
    value: 5
{% endhighlight %}

Moreover, as you can see, the value of `jwt_expiration_time` has been
simplified. Now, instead of `"5.minutes"` you have to specify it like this
`5`. The expiration time is now represented in minutes, so it no longer needs
to be specified explicitly. The `"5.minutes"` format is still supported, but this
support will be removed in the future.

Besides all that, there have been lots of additions in the configuration that
you can check out [here](/docs/Configuring-Portus.html).

### Upgrading the database

<div class="alert alert-info">
  <strong>Important note</strong>: before doing anything at all, make sure to
  backup the contents of the database.
</div>


With the new code in place, simply run the following (wrap it with `portusctl`
if you are using the RPM for openSUSE/SLE):

```
$ rake db:migrate
$ rake migrate:update_personal_namespaces
```

After that, if you are using an LDAP server for authentication, run the following:

```
$ rake migrate:update_ldap_names
```

The previous task will list the users which are to be updated, and then
it will ask you whether you want to proceed or not. If you don't want this task
to ask you this question (make it non-interactive), simply set the
`PORTUS_FORCE_LDAP_NAME_UPDATE` environment variable before running the rake
task (Note that it will only check the existence of the environment variable,
not its value).

Finally, as an optional step, you might want to execute the following task:

```
$ rake portus:update_tags
```

This task will fill the database with the digest and the image ID of all the
tags stored in the database. This will take a lot of time if you have lots of
images and tags in your Portus instance. For this reason, we recommend setting
the registry to `readonly` mode before performing this operation, just to avoid
any concurrency issues. Moreover, this task also asks for confirmation before
doing anything at all. You can skip this by setting the
`PORTUS_FORCE_DIGEST_UPDATE` environment variable before calling the rake task.