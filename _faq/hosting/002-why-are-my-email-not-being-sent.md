---
title: Why are my emails not being sent?
slug: why-email-not-sent
layout: faq
category: hosting
tags: [troubleshoot]
permalink: /faq/hosting/:slug
date: 2018-03-14 00:00:00 Z
---

This can come from a variety of reasons, here are the most common ones.

### Reason 1: Configuration issues

There may be an issue with some of the [SMTP configuration](/configure/email/setup)
items, such as credentials, or the hostname, or the port for the selected protocol.

By default passbolt is quite discrete on why a given configuration is not working. You can use the following
command to send a test email and get more debug information:

```shell
$ ./bin/cake passbolt send_test_email --recipient=youremail@domain.com
```

{% include messages/notice.html
    content="<b>Notice:</b> This command is run either from <code>/var/www/passbolt</code> or, if you installed passbolt using the Debian package or
    are using the passbolt VM (OVA): <code>/usr/share/php/passbolt</code>."
%}

If this fails you should double check what is the recommended configuration in your email provider documentation.
You can also ask on the community forum in case another user have a working configuration for the same provider.

### Reason 2: Email notifications are disabled in the config

Another reason could be because email notifications are disabled in you configuration.
You can review such settings in the administration panel, when you are logged in as an administrator in passbolt.

{% include articles/figure.html
    url="/assets/img/help/2019/05/AD_email_notification_send_settings.png"
    legend="Email Notification Settings - Email Delivery"
%}

To find the specific mail-related configurations set during setup, you will need to look
in `/var/www/passbolt/config/passbolt.php`. If you used the debian package to install passbolt
look in `/etc/passbolt/passbolt.php`.
If you are using the Docker version, you'll need to review your environment variables in `env/passbolt.env`.

### Reason 3: The cron job to send email is missing

Passbolt uses a system of email queue to send email notifications.
In practice this means there is a dedicated job that runs to go through the queue and send
emails at the interval of your choice.

So if you manage to send the test email but are not receiving notifications (such as registration emails),
one of the reason may be that the cron job required to send the emails is missing.
Depending on the method you used to install, it may have not be set, or failed to set.

In order to find out you should check if you have the following cron for the web server user
(`www-data` on Debian/Ubuntu, `nginx` on Centos), running where you passbolt was installed
(`/var/www/passbolt` by default).
```bash
$ crontab -u www-data -e
* * * * * /var/www/passbolt/bin/cron >> /var/log/passbolt.log
```

{% include messages/notice.html
    content="<b>Notice:</b> For installs from the Debian package or
    if you are using the passbolt VM (OVA), the cron job is automatically installed
    at <code>/etc/cron.d/passbolt-{ce|pro}-server</code>"
%}

If the cron is present, you should check if there are more data in the `/var/log/passbolt.log` log file,
it may give you more information about the issue.

### Reason 4: The webserver user is not allowed to run cron jobs

On some system, like RHEL or Centos, the`nginx` user may not have the right to run the cron job to send emails,
because of your PAM configuration. In this case you may want to add `nginx` to the PAM config to have `cron`
and `crond:local` permissions.

### Reason 5: There is an issue with the database schema related to the email queue

If after an update you are getting error messages such as:
```
Exception: SQLSTATE[42S22]: Column not found: 1054 Unknown column ‘EmailQueue.to’ in ‘field list’ ...
```

It is possible that the wrong version of the data model is stored in the cache. This can happen
if the cache is not cleared after an install or an update. You can try clearing out the cache to solve this.

```
./bin/cake cache clear_all
```

### Reason 6: You can manually send emails but cron job is not sending them

You may have a file permissions issue - for example, maybe you are running commands as root or files became
owned only by root. You can reference [this page](/hosting/update/install-scripts.html) and look for the
section named "Make sure the permissions are right for your current user". The commands found in that section
are useful for when you are updating your passbolt, and may also be helpful to resolve your mail issues.

### Reason 7: Gmail SMTP-Relay not accepting 'localhost' as EHLO

If you are using Google's G-Suite SMTP Relay, you need to ensure you have set your public IP address of your passbolt server in the  configuration.

Non-Docker: 
```
'EmailTransport' => [
    'client' => 'ip.add.re.ss'
]
```
Docker : 
```
EMAIL_TRANSPORT_DEFAULT_CLIENT= 'ip.add.re.ss'
```
For more information, see [this post](https://community.passbolt.com/t/email-not-working-smtp-relay-gmail/3281).
