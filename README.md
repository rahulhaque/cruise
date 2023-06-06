# Cruise for Laravel

A highly opinionated Laravel Sail like implementation of docker that focuses more on freedom of the development environment rather than sticking to your per project docker configuration. This is a setup once and use anytime on any machine type approach to give the developer the portability of their development environment. Cruise assumes you've no development tools installed in your system except docker and will provide all the tools in a bundled together portable environment.

## Requirements

- Docker Desktop

> If you're not running Docker Engine instead of Docker Desktop for some reason, make sure to update the configs of host's MySQL, Redis, PosgreSQL etc. to listen to (bind to) `0.0.0.0` instead of `127.0.0.1`. Otherwise you won't get access to host's database from within the container. Also, make sure to `CREATE USER 'root'@'%'` and `GRANT ALL ON *.* TO 'root'@'%'';` for database access. If you've installed Docker Desktop, you don't need any of the above steps.

## Installation

```bash
# Download or clone the repo in your home
# directory inside `~/.cruise` folder
git clone https://github.com/rahulhaque/laravel-cruise.git ~/.cruise

# Install cruise by running `install` command
~/.cruise/cruise install

# Run `cruise` to see available
# commands and options
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

Get inside the project directory and edit the `.env` environment file to redirect local resource calls from docker to your PC by replacing `127.0.0.1` with `host.docker.internal`. For example -
```bash
...
DB_HOST=host.docker.internal
...
REDIS_HOST=host.docker.internal
...
MAIL_HOST=host.docker.internal
...
```

This will allow `cruise` to connect to your local MySQL, Redis or Mailpit setup from the container. Now follow the next steps for the basics of `cruise`.

```bash
# Open terminal in the project directory
# Run project with Nginx in background
# Visit `http://localhost`
cruise start -b

# Or
# Run project with Nginx + Octane in background
# Visit `http://localhost`
cruise start -s octane -b

# Or
# Define port to run on
# Visit `http://localhost:8080`
cruise start -s octane -b -p 8080

# Run the same command again or `cruise shell`
# to drop inside the running project and
# use it however you like
cruise shell

# Logout from the shell
# or press `ctrl+d`
logout

# Stop the running project keeping
# the changes in the container
cruise stop

# Stop the running project discarding the
# changes and removing the container
cruise stop -f
```

#### Vite Configuration

If you plan to use Laravel Mix, Vite, Inertia, Vue or React, make sure to expose related ports with `-e <local_port>:<container_port>` option so that they can be accessible. Disable any ad-blocker on local development sites as they sometime block vite server and throws `net::ERR_BLOCKED_BY_CLIENT` error.

```bash
# Vite users start the project with
cruise start -b -e 5173:5173

# Edit the `vite.config.js` and
# add the following in the root
export default defineConfig({
    server: {
        host: '0.0.0.0'
    },
    ...
}

# Drop inside the running project
# with `cruise shell` and
# start the vite server
npm run dev
```

If you've defined a version when starting a project with `-v` option, like - `cruise start -v 7.4 -b`. Then you must also do the same for running other Cruise commands as well for that project, such as - `cruise shell -v 7.4` or `cruise stop -v 7.4`.

### 3. Other Developments

You can use Cruise to do any development of your choice as long as the included tools support. Simply run `cruise shell` anywhere to get the temporary development shell up with all the tools available to you. You can also start the server from shell mode. Just run `start-server` inside the shell and it will start PHP-FPM and Nginx for you. By default no port is exposed for temporary shell session. To bind any available port from your PC to any port inside the shell, run `cruise shell` with `-e` option, such as - `cruise shell -e 8080:80`. While Cruise provides out of the box support for Laravel applications, it is not bound to only Laravel development. Fork it, customize it and use it however you like.

> **Note:** Any command unknown to Cruise will be passed to Docker.

#### React Development Example

A basic react application creation and setup process example with Cruise is given below -

```bash
# Create the react project directory and open
# a terminal inside the project directory
# Run this to start a cruise shell
cruise shell -b -e 3000:3000
# `-b` to keep the container for reuse
# `-e` to open the ports for React

# Run the same command again or `cruise shell`
# to drop inside the running shell
cruise shell

# Create your react project
npx create-react-app .

# Run the app with `npm start`
# Visit `http://localhost:3000`
npm start

# Logout from the shell
# or press `ctrl+d`
# after done
logout

# Stop the running project keeping
# the changes in the container
cruise stop

# Stop the running project discarding the
# changes and removing the container
cruise stop -f
```

## Customization

A very good default is provided out of the box to handle most Laravel applications and PHP projects. If you still need to change anything, configure the dockerfile according to your need. There are two configurable docker environments provided by default. Both can run regular Laravel application with both Nginx and Octane. They include the followings -

- Ubuntu 22.04
- Git
- PHP
- Composer
- Nginx
- Node
- Npm
- MS Fonts
- Wkhtmltopdf (qt patched)
- Wkhtmltoimage (qt patched)
- Oh-my-zsh with auto-suggestions plugin

Edit the dockerfile in `~/.cruise/environments` directory if default is not what you're looking for. Then rebuild the image again with `-f` option like - `cruise build -v 8.1 -f`.

## Backstory

Well, I started off with Cruise being a highly opinionated implementation of those of like Laravel Sail. So, what are they? Why I thought of making Cruise and what problems it solves? It all started with me being forced to switch my workstation/laptop multiple times within a month or so because of technical issues. Installing and configuring everything from scratch every time for my development was time wasting and distracting. I wanted my development tools to be - **configured**, **backed up**, **OS independent**, **portable** and **accessible offline**. While docker can give me all of the above, it comes with the sheer complexity of its own. So, I started making Cruise and tried to make it as simple as possible for both expert and beginners in docker.

- Cruise makes the most uses of PC's local resources, like - MySQL, Redis, PostgreSQL, Mailpit etc. for everything instead of project specific multiple database, cache, mail containers. More containers, more things to manage, introduces more complexity.
- I wanted my frequently used development tools within hand's reach, such as - PHP, Composer, Node, Npm, Git, Nginx along with different version available any time anywhere. Cruise can give you a shell with all of the above anywhere any moment with a simple command either for some quick tinkering, or long term persistent tasks, you decide.
- Cruise focuses on configuring the base once and then use it everywhere without thinking about if one tool available in one container is available in another. Saving time in configuring project wise docker settings.
- Cruise aims to simplify using docker while not hiding all the magic happening behind. It helps in learning docker for new comers by showing all the commands executed in the background.
- You can create a backup of your configured environment without being too specific to your project and carry it around in pen portable drives. Usable even if there is no internet connection.
- Last but not the least, Laravel Sail uses `php artisan serve` to serve the application in a single threaded environment. Cruise enables multi-threading and runs the application from real world perspective with PHP-FPM and Nginx.

Lastly, this may not be the best solution for you based on how you like to manage your projects. Maybe you are comfortable in some other set up. If you happen to try this out, please, do share any idea, recommendation or feedback.

## Credits

-   [Rahul Haque](https://github.com/rahulhaque)
-   [All Contributors](../../contributors)
