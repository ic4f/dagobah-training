layout: true
class: inverse, middle, large

---
class: special
# Extending basic installation

slides by @dblankenberg @abdulrahmanazab

.footnote[\#usegalaxy / @galaxyproject]

---

class: normal
# Common `galaxy.yml` Tweaks

---

class: normal
# Configuring uWSGI

In `galaxy.yml`:
* `uwsgi:` section contains basic internal HTTP server configuration
  * `http: 127.0.0.1:8080` change to the port and to IP of listening network interface, or `0.0.0.0` for all network interfaces.
  * `threads: 4` change the number of threads for each web server process.
  * `offload-thread: 2` change the number of threads for serving static content.

---

class: normal
# Configuring the Galaxy Application

In `galaxy.yml`:
* `galaxy:` section contains application configuration
  * `filter-with: gzip` uncomment to enable internal gzip compression.
  * Uncomment `filter-with: proxy-prefix` and set `prefix` in `[server:]`if running Galaxy away from webserver root.
  * If running more than 1 Galaxy, set `cookie_path` to match `prefix`.
  * `database_connection` contains the SQLAlchemy connection string for database.

---

class: normal
# More Authentication Options

In `galaxy.yml`:
* `require_login: False` Force everyone to log in (disable anonymous access).
* `allow_user_creation: True` Allow unregistered users to create new accounts (otherwise, they will have to be created by an admin).
* `allow_user_deletion: False` Allow administrators to delete accounts.
* `allow_user_impersonation: False` Allow administrators to log in as other users (useful for debugging)
* And more ...

---

class: normal
# Securing your Object IDs

In `galaxy.yml`:
* User facing object IDs
  * Galaxy uses the Blowfish cipher to obscure integer IDs.
  * Prevents guessing the hashes for various Galaxy objects.
  * `id_secret` is used as the Blowfish key.
  * Changing `id_secret` is highly recommended.
  * Changing `id_secret` will change & invalid existing URLs (e.g. datasets, histories, workflows, etc).
* Try changing the `id_secret`. What happens?

---

class: normal
# Customizing your "Brand"

In `galaxy.yml`:
* `brand` appends "/{brand}" to the "Galaxy" text in the masthead.
* `logo_url` sets the URL linked by the "Galaxy/brand" text.
* `wiki_url` sets the URL linked by the "Wiki" link in the "Help" menu.
* `support_url` sets the URL linked by the "Support" link in the "Help" menu.
* `citation_url` sets the URL linked by the "How to Cite Galaxy" link in the "Help" menu.
* `search_url` sets the URL linked by the "Search" link in the "Help" menu.
* `mailing_lists_url` sets the URL linked by the "Mailing Lists" link in the "Help" menu.
* `screencasts_url` sets the URL linked by the "Videos" link in the "Help" menu.
* `terms_url` sets the URL linked by the "Terms and Conditions".

---

class: normal
# Adding a Notice box

In `galaxy.yml`:
```
message_box_visible: True
message_box_content: Downtime scheduled Sunday at Noon to one.
message_box_class: warning
```

---

class: normal
# Update the welcome page

Welcome page is `$GALAXY_ROOT/static/welcome.html` and is the first thing that
users see. It is a good idea to extend it with things like:
- Downtimes/Maintenance periods
- New tools
- Publications relating to your Galaxy

No restarting is necessary.

---

class: normal
# Client Browser Security

In `galaxy.yml`:
* `sanitize_all_html` by default, Galaxy sanitizes all "text/html" tool outputs. Setting to False potentially exposes users to XSS attacks.
* `sanitize_whitelist_file` manually override html sanitization for listed tools. Can set in admin interface.
* `serve_xss_vulnerable_mimetypes` certain filetypes (e.g. SVG) can contain JS that is vulnerable to XSS (Cross-site scripting) and are served as "plain/text" by default.
* `allowed_origin_hostnames` Returns Access-Control-Allow-Origin response header that matches the Origin header of the request.
* `use_printdebug` anything "print"ed within a Galaxy Web thread is exposed to user. Set to False in production.
* `use_interactive` Enabled by default. Enable live debugging in your browser.  This should *never* be enabled on a public site.

---

class: normal
# Configuring FTP

In `galaxy.yml`:
* `ftp_upload_dir` directory containing subdirectories matching users' identifier (defaults to e-mail)
* `ftp_upload_site` hostname of your FTP server provided to users in the help text. Must set to enable FTP import.
* `ftp_upload_dir_identifier` user attribute for subdirectory naming. 'email' (${user.email}) is default, but 'id' or 'username' are also common.
* `ftp_upload_dir_template` Python string template used to determine a users FTP upload directory (${ftp_upload_dir}/${ftp_upload_dir_identifier})
* `ftp_upload_purge` delete files after FTP import (True)

---

class: normal
# Exercise

* [Exercise - Configuring FTP](https://github.com/galaxyproject/dagobah-training/blob/2018-oslo/sessions/06-extending-installation/ex1-proftpd.md)

---

class: normal
# Configuring SMTP

In `galaxy.yml`:
* SMTP server
  * `smtp_server` host:port of SMTP server to use. Uses STARTTLS, but will fallback.
  * `smtp_username` username for SMTP server
  * `smtp_password` password for SMTP server. STARTTLS recommended on SMTP server.
  * `smtp_ssl` if SMTP server requires SSL from connection start, set to True.
* Addresses
  * `error_email_to` address to send user error reports to.
  * `email_from` return address used in automatic user notifications. (<galaxy-no-reply@HOSTNAME>)
  * `mailing_join_addr` mailing list to subscribe users to during registration.

---

class: normal
# Exercise: Configuring SMTP for Bug Reports

Install a mail tranfer agent (MTA):
* Here is a tutorial on how to [install Postfix MTA](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-as-a-send-only-smtp-server-on-ubuntu-14-04)

In `galaxy.yml`:
* set `smtp_server: localhost:25`
* set `error_email_to: you@email.tld` (use your email)
* Run a Galaxy tool the produces an error
* attempt to send a bug report

---

class: normal
# Configuring Data Library Path Uploads

In `galaxy.yml`:
* Admins Only
  * `library_import_dir` upload files from configured directory.
  * `allow_library_path_paste` set to True to allow admins to paste any path to upload.
* Users
  * `user_library_import_dir` root directory containing sub-directories named by user emails. A common setup is that the value for `user_library_import_dir` is the same as for `ftp_upload_dir` allowing users to upload files via FTP and then import them either in history or data library.
* Disabling compressed library archives downloads
  * `disable_library_comptypes` Can be 'zip', 'gz', and 'bz2'

---

class: normal
# Integrating Custom Biostars site

[Biostars](https://www.biostars.org) is a forum software for Q&A in life sciences
In `galaxy.yml`:
* `biostar_key` shared key with Biostar instance.
* `biostar_key_name` Cookie parameter name for storing shared key.
* `biostar_enable_bug_reports` (False) allow posting bug reports to Biostars (instead of just email)
* `biostar_url` URL to Biostars instance (must share base URL with Galaxy, e.g. biostar.usegalaxy.org)

---

class: normal
# galaxy.yml Explained

Everything You Always Wanted to Know About `galaxy.yml`\* (\*But Were Afraid to Ask):
* https://raw.githubusercontent.com/galaxyproject/galaxy/dev/config/galaxy.yml.sample

 ---
