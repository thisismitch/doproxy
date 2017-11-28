# doproxy

A Ruby script to scale HTTP service from an HAProxy server (DigitalOcean droplet). Use it on an Ubuntu 16.04 DigitalOcean droplet that is running HAProxy 1.6.

It can:

- Create new droplets that are running an HTTP service, and add those droplets to a backend of HAProxy
- Delete the droplets from the backend, and the droplets themselves

Currently, the actions are manually-initiated via a command line interface. If you wanted to get creative, you could use a monitoring system (like Nagios or Icinga) to trigger doproxy scaling based on resource utilization thresholds.

The choice of using a basic inventory file was intentional, to provide a simple example of using the DigitalOcean API, cloud-config, and DropletKit (DigitalOcean API gem) in a semi-useful way.

## Installation

Create an Ubuntu 16.04 Droplet on DigitalOcean. The Droplet will need private networking.

[Install HAProxy 1.6](https://www.digitalocean.com/community/tutorials/how-to-implement-ssl-termination-with-haproxy-on-ubuntu-14-04#install-haproxy-16x) on it:

```
sudo apt-get update
sudo apt-get install haproxy
```

Install rbenv prerequisites:

```
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev git
```

As **root**, the user that will be running doproxy, install Ruby 2.4.x:

```
cd
git clone git://github.com/sstephenson/rbenv.git .rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL

git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
exec $SHELL

rbenv install 2.4.2
rbenv global 2.4.2
```

As root, install [DigitalOcean DropletKit Gem](https://github.com/digitalocean/droplet_kit):

```
gem install droplet_kit
```

Then clone the repo to a directory of your choice:

```
git clone https://github.com/thisismitch/doproxy.git
cd doproxy
```

## Configuration

You'll need to generate a **read and write** access token in Digital Ocean's control panel at [https://cloud.digitalocean.com/settings/applications](https://cloud.digitalocean.com/settings/applications).

You will probably want to also look up the IDs or fingerprints of the SSH keys that you want to add to the droplets that doproxy will create. You can do this via [the DigitalOcean control panel](https://cloud.digitalocean.com/settings/security) or via the API.

### doproxy config

Copy configuration sample:

```
cp doproxy.yml.sample doproxy.yml
```

In the `doproxy.yml` file, set the following:

- Replace `token` value
- Add your own `ssh_key_ids`

You may also want to change any of the other options, such as `region` (which must be the same as the droplet that doproxy is installed on), `size`, and `image`.

Feel free to modify the HAProxy config template with your own values.

### Userdata

The default userdata file, which was written for use with Ubuntu 16.04 images, simply installs Nginx and replaces its index.html file with its hostname, public ip address, and droplet id (using the [DigitalOcean Metadata](https://www.digitalocean.com/community/tutorials/an-introduction-to-droplet-metadata) service). This is only for demonstration, and probably isn't useful for you. Replace it with something that will install your application.

If you want to use the sample userdata, copy it:

```
cp user-data.yml.sample user-data.yml
```

## Usage Examples

Print the usage screen:

```
ruby doproxy.rb
```

Print backend inventory:

```
ruby doproxy.rb print
```

Create a new droplet and add it to the load balancer:

```
ruby doproxy.rb create
```

Delete droplet 5:

```
ruby doproxy.rb delete 5
```

## Known Issues and Assumptions

Issues:

- There are tons of error cases that aren't being checked
- Inventory file should be replaced with a different system that is more resilient
- Creating a droplet blocks input until it is active (this was done intentionally, but can be modified to just reconfigure HAProxy immediately if additional error checking is added)

Assumptions

- API token has read and write permission, and is not hitting a rate-limit
- Inventory file exists (hint: `touch inventory`)
- You are using a region that supports user-data
- Assumes that a droplet create will succeed (droplet will turn to "active" status)
- The HAProxy backend names will not match the hostname, as listed in DigitalOcean's control panel. This could be fixed be renaming the droplet after it is created
