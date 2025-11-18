#odoo
start the odoo server with basic options

```powershell
python odoo-bin -r dbuser -w dbpassword --addons-path=addons -d mydb
```

start the odoo server with addons

```powershell
 uv run python odoo-bin -r odoo -w odoo --addons-path=C:\Users\CBELLO\sources\repos\odoo\odoo-19.0\odoo\addons,C:\Users\CBELLO\sources\repos\odoo\odoo-19.0\addons -d odoo --with-demo
```

### odoo confif file

```conf
[options]
admin_passwd = 01234
;logfile =  C:\Users\CBELLO\sources\repos\odoo\odoo-log.log
log_level = debug
db_host = localhost
db_user = odoo
db_port = 5432
db_password = odoo
max_cron_threads = 1
workers = 10
addons_path = C:\Users\CBELLO\sources\repos\odoo\odoo-19.0\addons,C:\Users\CBELLO\sources\repos\odoo\odoo-19.0\odoo\addons,C:\Users\CBELLO\sources\repos\odoo\odoo-19.0\custom-addons
```