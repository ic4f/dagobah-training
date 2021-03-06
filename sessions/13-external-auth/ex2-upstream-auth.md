![galaxy logo](../../docs/shared-images/galaxy_logo_25percent_transparent.png)

### Galaxy Administrators Course

# Upstream Authentication - Exercise

#### Authors: Nate Coraor (2017), Nicola Soranzo (2018)

## Learning Outcomes

By the end of this session you should:
1. be familiar with configuring Galaxy to use an upstream (proxy) authentication provider
2. be able to log in to your Galaxy server with a file-configured user.

## Introduction

For this exercise we will use a basic password file method for authenticating - this is probably not a very useful method in production, but it demonstrates how the proxy server can be configured to provide the correct header to Galaxy, and how Galaxy integrates with upstream authentication providers.

## Section 1 - Configure nginx

**Part 1 - Configure nginx**

Begin by editing `/etc/nginx/sites-available/galaxy` (as root) and adding the `auth_basic`, `auth_basic_user_file` and `uwsgi_param` directives to the `location / { ... }` block as below:

```nginx
    location / {
        #proxy_pass          http://galaxy;
        #proxy_set_header    X-Forwarded-Host $host;
        #proxy_set_header    X-Forwarded-For  $proxy_add_x_forwarded_for;
        uwsgi_pass           127.0.0.1:4001;
        include              uwsgi_params;
        auth_basic           galaxy;
        auth_basic_user_file /etc/nginx/passwd;
        uwsgi_param          HTTP_REMOTE_USER $remote_user;
    }
```

`auth_basic` enables validation of username and password using the "HTTP Basic Authentication" protocol. Its value `galaxy` is used as a realm name to be displayed to the user when prompting for credentials.

`auth_basic_user_file` specifies the file that keeps usernames and passwords, in the following format:

```
# comment
name1:password1
name2:password2:comment
name3:password3
```

`uwsgi_param` adds `HTTP_REMOTE_USER` to the special variables passed by nginx to uwsgi, with value `$remote_user`, which is a nginx embedded variable containing the username supplied with the Basic authentication.

**Part 2 - Create passwd file**

You can use the `openssl passwd` command to do this (replace `nate` with a username of your choosing and `qwerty` with a password):

```console
$ echo "nate:$(openssl passwd qwerty)" | sudo tee /etc/nginx/passwd
```

## Section 2 - Configure Galaxy

Galaxy needs to be instructed to expect authentication to come from the upstream proxy. In order to do this, set the following two options in `galaxy.ini`:

```ini
use_remote_user = True
remote_user_maildomain = example.org
```

Set the `remote_user_maildomain` option to the appropriate domain name for your site. Then, restart Galaxy.

If you visit your Galaxy server now, you should see the following message:

![access denied message](images/access_denied.png)

This is because we have not yet restarted nginx. Galaxy expects the `REMOTE_USER` header to be set by nginx. If it is not, Galaxy will refuse to allow access to its UI.

## Section 3 - Test

**Part 1 - Test**

Restart nginx with `sudo systemctl restart nginx`. You should now be presented with a password dialog when attempting to load the Galaxy UI.

Log in using the username and password you provided when creating the `passwd` file. If your username and the value of `remote_user_maildomain` match an existing user, you will be logged in to that account. If not, a new account will be created for that user.

Note that some user features are not available when remote user support is enabled.

Try logging out by selecting **User** -> **Logout**. You will discover that when returning to the user interface, you are still logged in. This is because Galaxy has no way of logging you out of the proxy's authentication system. Instead, you should set `remote_user_logout_href` in `galaxy.ini` to point to the URL of your authentication system's logout page.

**Part 2 - Undo changes**

We don't want to leave Galaxy this way for the rest of our workshop. Undo the changes by commenting `use_remote_user` from `galaxy.ini` and restarting Galaxy, and by commenting the options we added to `/etc/nginx/sites-available/galaxy` and restarting nginx.
