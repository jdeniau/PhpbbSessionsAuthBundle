# phpBB Sessions Auth Bundle

This allows you to use phpBB as a authentication provider and share its sessions.

It was originally developed for use on the new phpBB Symfony Website but was then open sourced.


## Configuration

First of all, make sure in your application to ignore the phpBB tables, by using (This is needed for each entity manager):

```yaml
doctrine:
    dbal:
        schema_filter: ~^(?!phpbb_)~
```        
where phpbb_ is your table prefix for tables generated by phpBB. Not making this configuration change can cause your forum tables to be deleted!

Then add a custom entity_manager and dbal to your doctrine connections:
```yaml
doctrine:
    dbal:
        default_connection: default
        connections:
            acme: # this is an example for your website's database but it's not required
                driver:   "%database_driver%"
                host:     "%database_host%"
                port:     "%database_port%"
                dbname:   "%database_name%"
                user:     "%database_user%"
                password: "%database_password%"
                charset:  "UTF8"
            forum:
                driver:   "%forum_database_driver%"
                host:     "%forum_database_host%"
                port:     "%forum_database_port%"
                dbname:   "%forum_database_name%"
                user:     "%forum_database_user%"
                password: "%forum_database_password%"
                charset:  "UTF8"

    orm:
        entity_managers:
            default: # same here, not required, but you will probably have this in your configuration
                connection: default 
            forum:
                connection: forum
                mappings:
                    phpbbSessionsAuthBundle: ~
```

Then add this in your `app/config/config.yml` file. This is the bundle configuration
```yaml
phpbb_sessions_auth:
    session:
        secure: false # set to true this if you use secure cookie in your php admin configuration
        cookiename: "phpbb_foo" # must match your forum admin cookie name configuration
        boardurl: "/forum/" # your phpbb url
        force_login: false # if true, anonymous users will be redirected to the login page
    database:
        entity_manager: "forum" # must match the key bellow doctrine.orm.entity_managers
        prefix: "phpbb_" # change this if you do not use the default "phpbb_" prefix
```

Update your `app/config/security.yml` file to match this:
```yaml
security:
    providers:
        phpbb: 
            id: "phpbb.sessionsauthbundle.phpbb_user_provider"
    firewalls:
        main:
            pattern: ^/
            anonymous: true
            stateless: true # stateless should be set to true, or your symfony user may be stored in the session even if you logged out from the phpbb instance 
            form_login:
                login_path: /forum/ucp.php?mode=login # this should theorically not be set, but if not returns a 500 instead a 403, see https://github.com/symfony/symfony/issues/8467 for more informations
            simple_preauth:
                authenticator: "phpbb.sessionsauthbundle.phpbb_authenticator"
            provider: phpbb
```

## Missing functionality

There are some few edge functionality missing: 

  * `"Remember me" key expiration length (in days)` (ie. max_autologin_time) is not used, and thus if this number is grater than 1, the user will not be autamatically logged out unless he goes to the forum.
  * `Session IP validation` is considered as "A.B.C", no matter what you specified in your Admin Control Panel configuration

## Other possible improvements
The following configuration points could be fetch in the `phpbb_config` database :
  * `secure` = `cookie_secure`
  * `cookiename` = `cookie_name`

## Confession

This is taken from (https://github.com/jewome62/PhpbbSessionsAuthBundle) and used in a
private project. I needed a version compatible with symfony 3.x.
