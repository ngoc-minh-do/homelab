# Nextcloud

## Issue

`WebUI > Overview > Security & setup warnings` is same as result from `sudo -u www-data php occ setupchecks`

1. `WebUI > Overview > Security & setup warnings` show
```
One or more mimetype migrations are available. Occasionally new mimetypes are added to better handle certain file types. Migrating the mimetypes take a long time on larger instances so this is not done automatically during upgrades. Use the command `occ maintenance:repair --include-expensive` to perform the migrations.
```
    On `nextcloud-aio-nextcloud` container, run:
```
sudo -u www-data php occ maintenance:repair --include-expensive
```
2. `WebUI > Overview > Security & setup warnings` show
```
Could not check that the data directory is protected. Please check manually that your server does not allow access to the data directory. To allow this check to run you have to make sure that your Web server can connect to itself. Therefore it must be able to resolve and connect to at least one of its `trusted_domains` or the `overwrite.cli.url`. This failure may be the result of a server-side DNS mismatch or outbound firewall rule.
```
    Reason could be it failed to resolve domain name inside container.
    Confirm by
```
`curl -i -X PROPFIND https://my.domain.org/remote.php/webdav`
`nslookup my.domain.org`
```
3. WebUI's Logging show `"xxx" is locked, existing lock on file: exclusive"`

    On `nextcloud-aio-nextcloud` container, get db's information from `config/config.php`, then run:
```
psql -h <server> -U <user> -d <database>
DELETE FROM oc_file_locks WHERE true 
```
Then, run:
```
sudo -u www-data php occ files:scan --all
```
4. To test if Nextcloud client is using `notify_push`
```
sudo -u www-data php occ notify_push:metrics
sudo -u www-data php occ notification:test-push <user-id>
sudo -u www-data php occ notification:generate <user-id> <message>
Then check notification on mobile app
```