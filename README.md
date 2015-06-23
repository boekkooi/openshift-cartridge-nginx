# Openshift Nginx Cartridge
Welcome to a life where [nginx](http://nginx.org/) is possible on [openshift](https://www.openshift.com/).
 
This cartridge allow you to create a scalable nginx application.
Combine this with the [boekkooi PHP cartridge](https://github.com/boekkooi/openshift-cartridge-php) and you have a scalable application using the latest versions.

Just create your app using:

	$ rhc create-app <yourapp> http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx

If you want to install a specific nginx version you can add `--env OPENSHIFT_NGINX_VERSION=<version>` to the command.
For example to install nginx 1.9.1 you can use:

	$ rhc create-app <yourapp> --env OPENSHIFT_NGINX_VERSION=1.9.1 http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx

## Build a scalable wordpress
##### Many thanks to  :sparkles:'boekkooi' :sparkles: for his excellent repos 
##### and also to  :sparkles:'wordpress-quickstart' :sparkles: repo
##### 1. To create an app:

	# create (if scalable with -s) nginx app with php cartridge
	$ rhc create-app <yourapp> --env OPENSHIFT_NGINX_VERSION=1.9.1 http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx -s --env OPENSHIFT_PHP_VERSION=5.6.6 http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-php
	# add database
	$ rhc cartridge add mysql-5.5 -a <yourapp>
	# **Only if you want to use Redis for Object cache**
	$ rhc cartridge add -a <yourapp> http://cartreflect-claytondev.rhcloud.com/reflect?github=ranib/openshift-redis-cart

##### 2. Follow PHP Configuration instructions [from](https://github.com/ranib/openshift-cartridge-php)
##### 3. I have already included all that in this repo
##### 4. To add Scalable-Wordpress:

	# **Only if you don't have local copy of <yourapp> created above**
	$ git clone <git-uri> <yourapp>
	# then change to <yourapp> dir
	$ cd <yourapp>
	$ git remote add upstream https://github.com/ranib/openshift-scalable-wordpress
	$ git pull upstream master

##### 5. edit wp-config.php and change 'true' to 'false' (no SSL) in the line below

	define('FORCE_SSL_ADMIN', true);

##### or add after the line (SSL will be limited to rhcloud domain, will have to get SSL Cert for custom domain)

	define(‘FORCE_SSL_LOGIN’, true);

	/** In Nginx, to resolve "You do not have sufficient permissions to access this page" error after http(s) SSL login */
	if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
		$_SERVER['HTTPS'] = 'on';

	/* That's all, stop editing! Happy blogging. */

##### 6. in wp-config.php file add (before the above line)

	/** Woocommerce plugin recommends to increase WP Memory Limit to 128MB */
	define( 'WP_MEMORY_LIMIT', '128M' );

	/** path to fastcgi cache directory for Nginx Helper plugin */
	define('RT_WP_NGINX_HELPER_CACHE_PATH', getenv('OPENSHIFT_TMP_DIR').'/nginx-cache');

##### 7. fix conflicts (if any) and edit (root directory) in action_hooks/deploy
##### may have to run this command to make action_hooks/build executable

	$ git update-index --chmod=+x --add .openshift/action_hooks/*

##### 8. then commit

	$ git add -A
	$ git commit -am 'install wordpress'
	$ git push

##### *This is a reminder for me to upload my fully configured/functional item 2, 3 & 5*

## Versions
Currently this cartridge has the following versions:
- 1.6.2
- 1.9.1

If you need another version you can compile it yourself and submit a PR to get it integrated.

### Compiling a new version
To compile a new version you will first need a openshift application.

	$ rhc create-app nginx http://cartreflect-claytondev.rhcloud.com/github/ranib/openshift-cartridge-nginx

Now clone the repository and create a `nginx` folder. Now copy the `usr/compile` directory from [this](https://github.com/ranib/openshift-cartridge-nginx) repository.
Now set the versions you need to compile in the `nginx/compile/versions` file. Make 'build' executable `git update-index --chmod=+x --add nginx/compile/*` 
Commit and push the application repository.
  
SSH into your app `rhc ssh nginx` and go to the compile folder `cd ${OPENSHIFT_REPO_DIR}/nginx/compile` and start compiling by running the following command:

	./all

Once compiling is done you can download the `nginx.tar.gz` and extract or copy `nginx-{version}` folder from you application via ftp 
and place them into the `openshift-cartridge-nginx/usr` folder. Last but not least edit the `openshift-cartridge-nginx/metadata/manifest.yml` 
and add the versions. Change permissions to make 'nginx' executable `git update-index --chmod=+x --add usr/nginx-{version}/sbin/nginx`
All done, just commit and push to your `openshift-cartridge-nginx` repo `git add -A` & `git commit -am 'update nginx-{version}'` & `git push` and use:

	http://cartreflect-claytondev.rhcloud.com/github/<user>/openshift-cartridge-nginx
