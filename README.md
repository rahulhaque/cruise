# Cruise 🚢

A highly opinionated Laravel Sail like implementation of docker that focuses more on freedom of the development environment rather than sticking to your per project docker configuration. This is a setup once and "use anytime on any machine" type approach to give the developers the portability of their development environment. Cruise assumes you've no development tools installed in your system except docker and will provide all the tools in a bundled together portable environment. To know more about the motivation behind **Cruise**, you can have a look at the [backstory](#backstory) first before diving in.

## Requirements

- Docker Desktop or Docker Engine

> If you've installed **Docker Desktop**, skip this part and go to [installation](#installation).
>
> If you're running **Docker Engine** (the one without GUI), make sure to update the configuration of **MySQL**, **Redis**, **PosgreSQL** etc. to listen to (bind to) `0.0.0.0` instead of `127.0.0.1`. Otherwise you won't get access to those from inside the container. Also, make sure to `CREATE USER 'root'@'%';` and `GRANT ALL ON *.* TO 'root'@'%';` for database access for **MySQL**. For **redis**, set `protected-mode no` in `/etc/redis/redis.conf`.

## Installation

```bash
# Download or clone the repo in your home
# directory inside `~/.cruise` folder
git clone https://github.com/rahulhaque/laravel-cruise.git ~/.cruise

# Install cruise by running `install` command
~/.cruise/cruise install

# Install traefik by running `install` command
~/.cruise/traefik install

# Run `cruise` to see available commands and options
# or press tab to see simple autocompletion
cruise
```

I suggest you go through the commands and options available for the first time to get a quick grasp of what is offered.

Bash and Oh-my-zsh auto completion is added by default. If you're using something else, source the completion script by adding the below lines.

```bash
if [ -f ~/.cruise/cruise-completion ]; then
    source ~/.cruise/cruise-completion
fi

if [ -f ~/.cruise/traefik-completion ]; then
    source ~/.cruise/traefik-completion
fi
```

Then run `omz reload` to load the new completion script.

## Building

After installing `cruise` binary, build the base image by running the following. This is a one time step for new install.

I recommend running `docker builder prune` command to clear any unnecessary build cache before building.

The Cruise environment versions are set to match major PHP versions for easy recall.

```bash
# Let's see which cruise environments are
# available for you to build by running
cruise environments

# Build the preferred environment with -v option 
cruise build -v 8.2
```

Check if the build is successful by running `cruise images`.

After successful build, it is recommended that you take a backup of the image before experimenting. If anything goes wrong, you have your stable offline backup image ready to roll. To create a backup -

```bash
cruise export -v 8.2
```

This will create a tar file of the backup in `~/.cruise/backup/cruise-8.2.tar`.

To import a backed up image, run the `import` command along with passing the path of the backup.

```bash
cruise import ~/.cruise/backup/cruise-8.2.tar
```

## Usage

I recommend using **Cruise** along with [Traefik](https://traefik.io/traefik/) proxy manager which will give you nice domain names for accessing your projects. If you don't know what Traefik is or heard the name for the first time, don't worry as I will guide you through the setup process. Install Traefik in a minute from this [gist](https://gist.github.com/rahulhaque/3801a0fbb5cb2b8f0f060a364f494cb7). Traefik will run in the background looking for any Cruise projects to be available to serve.

### 1. Creating New Laravel Project

```bash
# Open terminal and create project with
# Cruise will take the default version
# from `~/.cruise/.config` file
# See available environments
# with `cruise environments`
cruise create example-app

# To create for other PHP versions (i.e. 7.4)
cruise create -v 7.4 example-app
```

If you create a project with any version other than the default version, you will
have to use version `-b` option for later subsequent commands as **Cruise** keep the project folder clean by not making any config for itself. Another benefit of this is that you can switch and try different versions in an instant without touching the project files at all.

### 2. Running Existing Laravel Project

Get inside the project directory and edit the `.env` file to redirect container resource calls from docker to your PC by replacing `127.0.0.1` with `host.docker.internal`. For example -

```bash
...
DB_HOST=host.docker.internal
...
REDIS_HOST=host.docker.internal
...
MAIL_HOST=host.docker.internal
...
```

This will allow `cruise` to connect to your local MySQL, Redis or Mailpit setup from the container. Now follow along the next steps for the basics of `cruise`.

```bash
# Open a terminal in your project directory
# To run the project with Nginx and proxy through Traefik.
# If you have Traefik installed (recommended) and
# keep running it in the background (`-b`)
cruise traefik -b
# Visit `http://<your_project_name>.localhost`

# Or
# To run project with Nginx on specific port (`-p`)
# and keep running it in the background (`-b`)
cruise start -p 8000 -b
# Visit `http://localhost:8000`

# Or
# To run project with Nginx + Octane on specific
# port (`-p`) and server (`-s`) and keep
# running it in the background (`-b`)
cruise start -s octane -p 8000 -b
# Visit `http://localhost:8000`

# Without background (`-b`) option the project
# will stop as soon as you close the terminal

# Run the same command again or `cruise shell`
# to drop inside the running project and
# use it however you like
cruise shell
# `cruise shell` is an independent command.
# You can run it anywhere to spawn a
# temporary shell anytime

# Try out the environment by running
# `php -v`, `npm -v` etc. to
# see if you're really in
composer

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

If you plan to use Laravel Mix, Vite, Inertia, Vue or React, make sure to expose related ports with expose `-e <local_port>:<container_port>` option so that they can be accessible. Disable any ad-blocker on local development sites as they sometime block vite server and throws `net::ERR_BLOCKED_BY_CLIENT` error.

```bash
# Vite users can start the project by
# exposing (`-e`) required port and
# keep running it in the background
cruise start -e 5173:5173 -b

# Edit the `vite.config.js` and
# add the following in the root
# for access from host machine
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

### 3. Other Developments

You can use **Cruise** to do any development of your choice as long as the included tools support. Simply run `cruise shell` anywhere to get the temporary development shell up with all the tools available to you. You can also start the server from shell mode. Just run `start-server` inside the shell and it will start PHP-FPM and Nginx for you. By default no port is exposed for temporary shell session. To bind any available port from your PC to any port inside the shell, run `cruise shell` with expose (`-e`) option, such as - `cruise shell -e 8000:80`. While **Cruise** provides out of the box support for Laravel applications, it is not bound to only Laravel development. Fork it, customize it and use it however you like. See [customization](#customization) section for more.

> **Note:** Any command unknown to **Cruise** will be passed to **Docker**.

#### i. React Development Example

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

## VsCode Integration

There's high chance after running a project, you may want to use VsCode's remote desktop to code inside the docker container (use it as a dev container). To do this, just open VsCode, open all commands and look for `Dev Containers: Attach to running container`. After VsCode done installing its server, open all commands and look for `Dev Containers: Open Container Configuration File...` and paste the following.

```json
{
    "workspaceFolder": "/var/www/html",
    "remoteUser": "cruise"
}
```

## Customization

A very good default is provided out of the box to handle most Laravel, PHP, Node, React and Vue applications. Default environments include the followings -

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

You can edit the dockerfile in `~/.cruise/environments/<versions>` directories to customize the setup. Then rebuild the image with force (`-f`) option like - `cruise build -v <version> -f`.

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
