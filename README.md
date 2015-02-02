EngageNY.org
============

Vagrant for EngageNY.org

Setup
-----

1. Install Vagrant, Virtualbox, Git, and Drush

  Requires Vagrant 1.5+.

  See http://vagrant-up.com/downloads and https://www.virtualbox.org/wiki/Downloads for binary installers.
  See https://github.com/drush-ops/drush for info on installing drush on your system.

2. Highly suggested:

  Install vagrant-faster plugin.  This module will automatically scale up your vagrant box based on your host machine.

  ```
  vagrant plugin install vagrant-faster
  ```

  If you're on Mac OS X, you probably want to install homebrew (http://brew.sh/) because it makes installing ansible easy, and because it is awesome anyway.

  ```
  ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

  Then you can install ansible easily.

  ```
  brew update
  brew install ansible
  ```

2. Clone this repo if you haven't already.
  ````
  git clone git@github.com:pcgroup/engage-vagrant.git engageny
  cd engageny
  ````

3. Raise the virtual machine.
  ````
  vagrant up
  ````

  Vagrant will automatically clone the project repo to ./src on first up.

4.  Add this line to your hosts file to allow local.engageny.org to point to 1.2.3.4

  This Vagrantfile uses a local private network using the 1.2.3.4 IP address.

  ````
  # MacOS/Linux: /etc/hosts
  # Windows: C:\Windows\System32\drivers\etc\hosts
  1.2.3.4       local.engageny.org
  ````

5. Copy the database from another server. It is faster and more reliable to do this from within the vagrant box.

  You should test the drush connection first, and also accept adding the server to the Known Hosts. Otherwise, the first line of the prod.sql can be this Known Host question/response, which will give you a SQL error while trying to import. You should only have to do this the first time on a fresh vagrant.

  ```
  drush @engageny2.prod help
  ```

  Also, I find it more reliable to sql-dump then sqlc.

  ````
  vagrant ssh
  drush @engageny2.prod sql-dump > prod.sql
  drush @engage sqlc < prod.sql
  ````

  After syncing from production, you usually need to rebuild the registry:

  ```
  drush @engage rr
  ```

  Then, use `drush @engage upgradepath` to run all the things needed to rebuild the site.
  The command includes reverting all features, clearing all caches, and running all updates.

  It can take a long time the first time you sync from production if there are a lot of updates to do.

  You should also rebuild the search indexes to make sure they are up-to-date with the database you have loaded.

  ```
  drush @engage sapi-r
  drush @engage sapi-i
  ```

6. Visit http://local.engageny.org.
7. You can edit the files directly in `./src` (from your host machine) and they will be instantly reflected in the VM.

NOTES
-----

Drush aliases are created both on your host machine and inside the vagrant box called @engage.

By default, if you are on Linux/Mac OS X, the Vagrantfile will default you to NFS sharing as it is *much* faster than the default Virtualbox sharing.
On Windows, It will still use the default Virtualbox sharing. If you find this too slow, you can set up rsync on your machine by installing it through MSYS or Cygwin, and then change the lines

```
config.vm.synced_folder "src/#{vars['path_to_drupal']}", "/var/www",
  owner: "www-data", group: "www-data"
```
to
```
config.vm.synced_folder "src/#{vars['path_to_drupal']}", "/var/www",
  type: "rsync", rsync__exclude: ".git/"
```

Then you can use the `vagrant rsync` or `vagrant rsync-auto` to sync with the VM manually or on change, respectively.

You can also try the SMB sharing (https://docs.vagrantup.com/v2/synced-folders/smb.html), but I didn't have good luck with that the last time I tried it.