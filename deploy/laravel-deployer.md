## 1️⃣ Deployer Instalation

```bash
composer require lorisleiva/laravel-deployer
```

### Configuration
In order to generate your deployment configuration file, simply run:

```bash
php artisan deploy:init
```

## 2️⃣ Creating SSH keys
In order to create a new SSH key, run ssh-keygen -t rsa -b 4096 -m pem -f path/to/keyfile. This will prompt you for a key passphrase and save the key in path/to/keyfile.

Having a passphrase is a good thing, since it will keep the key encrypted on your disk. When configuring the secret SSH_PRIVATE_KEY value in your repository, however, you will need the private key unencrypted.


## Problem
```
➤ Executing task deploy:writable

  Unable to setup correct permissions for writable dirs.
  You need to configure sudo's sudoers files to not prompt for password,
  or setup correct permissions manually.
➤ Ex✔cuting task deploy:failed
✔ Executing task deploy:unlock

In writable.php line 95:

  Can't set writable dirs with ACL.
```
`
Solving: sudo apt-get install acl
`
Reference:
- https://github.com/atymic/deployer-php-action
- https://github.com/lorisleiva/laravel-deployer
- https://blog.logrocket.com/how-to-create-a-ci-cd-for-a-laravel-application-using-github-actions/
- https://atymic.dev/blog/github-actions-laravel-ci-cd/
