# Laravel Cruise

A highly opinionated Laravel Sail like implementation of docker that focuses more on freedom of the development environment rather than sticking to your per project docker configuration. This is a setup once and use anytime on any machine type approach to give the developer the portability of their development environment. Cruise assumes you've nothing installed in your system except docker and will provide everything in a bundled together portable environment.

## Requirements

- Docker

## Installation

```bash
# Download the repo in in your home
# directory `~/.cruise` folder
git clone https://github.com/rahulhaque/laravel-cruise.git ~/.cruise

# Install cruise by running `install` command
~/.cruise/cruise install

# Run `cruise` for available commands
cruise
```

## Building

After installing `cruise` binary, build the base image by running the following. This is a one time step for new install.

```bash
cruise build -v 8.1
```

Check if the build is successful by running `cruise images`.

After successful build, it is recommended that you take a backup of the image before experimenting. If anything goes wrong, you have your stable backup image ready to roll. To create a backup -

```bash
cruise export -v 8.1
```

This will create a tar file of the backup in `~/.cruise/backup/cruise-8.1.tar`.

To import a backed up image, run the `import` command along with passing the path of the backup.

```bash
cruise import ~/.cruise/backup/cruise-8.1.tar
```

## Usage

### 1. Creating New Laravel Project

```bash
# Open terminal in project install directory
cruise create example-app

# Create for available PHP versions (i.e. 7.4)
cruise create -v 7.4 example-app
```

### 2. Running Existing Laravel Project

```bash
# Open terminal in the project directory
# To start the project with - Nginx
cruise start -b

# To start the project with - Octane
cruise start -s octane -b

# Run the same command again or `cruise shell`
# to drop inside the running project and
# use it however you like
cruise shell

# Logout from the shell or press `ctrl+d`
logout

# Stop the running project while keeping
# the changes in the container
cruise stop

# Stop the running project discarding
# the changes in the container
cruise stop -f
```

### 3. Other Developments

You can use Cruise to do any development of your choice as long as the tools support. Simply run `cruise shell` anywhere to get the development shell up with all the tools available to you. While Cruise provides out of the box support for Laravel, it is not bound to only Laravel development. Use it to your heart's content.

## Customization

A very good default is provided out of the box to handle most Laravel applications and PHP projects. If you still need to change anything, configure the dockerfile according to your need. There are two configurable docker files provided by default. Both can run regular Laravel application with both Nginx and Octane. They include the followings -

- Git
- PHP
- Composer
- Nginx
- Node
- Npm
- MS Fonts
- Wkhtmltopdf (qt patched)
- Wkhtmltoimage (qt patched)
- Oh-my-zsh with necessary plugins

Edit the dockerfile in `~/.cruise/docker` directory if default is not what you're looking for. Then rebuild the image again with `-f` option like - `cruise build -v 8.1 -f`.

## Backstory

Well, I started off with Cruise being a highly opinionated implementation of those of like Laravel Sail. So, what are they? Why I thought of making Cruise and what problems it solves? It all started with me being forced to switch my workstation/laptop multiple times within a month or so because of technical issues. Installing and configuring everything from scratch every time for my development was time wasting and distracting. I wanted my development tools to be - **configured**, **backed up**, **OS independent**, **portable** and **accessible offline**. While docker can give me all of the above, it comes with the sheer complexity of its own. So, I started making Cruise and tried to make it as simple as possible for both expert and beginners in docker.

- Cruise makes the most uses of PC's local resources, like - MySQL, Redis, PostgreSQL, Mailhog etc. for everything instead of project specific multiple database, cache, mail containers. More containers, more things to manage, introduces more complexity.
- I wanted my frequently used development tools within hand's reach, such as - PHP, Composer, Node, Npm, Git, Nginx along with different version available any time anywhere. Cruise can give you a shell with all of the above anywhere any moment with a simple command either for some quick tinkering, or long term persistent tasks, you decide.
- Cruise focuses on configuring the base once and then use it everywhere without thinking about if one tool available in one container is available in another. Saving time in configuring project wise docker settings.
- Cruise aims to simplify using docker while not hiding all the magic happening behind. It helps in learning docker for new comers by showing all the commands executed in the background.
- You can create a backup of your configured environment without being too specific to your project and carry it around in pen portable drives. Usable even if there is no internet connection.
- Last but not the least, Laravel Sail uses `php artisan serve` to serve the application in a single threaded environment. Cruise enables multi-threading and runs the application from real world perspective with PHP-FPM and Nginx.

Lastly, this may not be the best solution for you based on your use case and project setup. Maybe you are comfortable in some other set up. If you happen to try this out, please, do share any idea, recommendation or feedback.

## Credits

-   [Rahul Haque](https://github.com/rahulhaque)
-   [All Contributors](../../contributors)
