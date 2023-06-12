# **Dependency Track SBOM Scans Setup (from scratch)**

Along this file I'll build a [Dependency Track](https://docs.dependencytrack.org/) server capable of consuming and producing [CycloneDX](https://cyclonedx.org/) [SBOM](https://www.cisa.gov/sbom)s of Node.js or PHP Composer applications to scan for vulnerabilities in theirs components. I'll also start writing a **script to automate** all these operations and reduce the gap between a new project to analyze and The Dependency Track analysis of the said project.
-

**NOTE:** This was all done using Ubuntu 22.04 with WSL2

# **Required softwares Installation**
We need some softwares in order to use others. 
Step by step we will see why and how we will install these softwares and programs.

- **NMV** Node version manager.
- **Git** THE famous version manager and source control software. (Thanks to Linus Torvalds)
- **Docker & Compose plugin** Dependency Track will be deployed in two containers for API server and frontend for easy and quick deployment.
- **PHP-cli & CycloneDX PHP Composer plugin** Plugin that creates CycloneDX formated SBOM for PHP Composer projects.


<details open><summary>Collapse if you already have them</summary>

## **NVM (Node Version Manager) Installation**
#### **Why ?**
We may need to install **one or more version of node** to met the requirements **for each projects** we may work on.

>`nvm` allows you to quickly install and use different versions of node via the command line.
>
> _Source: [nvm - github](https://github.com/nvm-sh/nvm)_.

#### **How ?**

First of all make sure you have `curl` installed.

```shell
$ curl --version
```


<details><summary>If not here is how</summary>

## Ubuntu / Debian

Use `apt install` to install the `curl` package from your distro's repositories.
```shell
$ sudo apt install curl
```
## Arch / Arch based

Use `pacman -S` to install the `curl` package from your distro's repositories.
```shell
$ sudo pacman -S curl
```

For other distributions look at [the following page](https://everything.curl.dev/get/linux).

</details>


#### **NVM Install & Update Script**

Check and change if needed the [latest version of nvm](https://github.com/nvm-sh/nvm/releases/latest) before running the following command.

```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
Then run `nvm -v` to check if it's installed and responding.

<details>
<summary>It should looks like this:</summary>

![nvm installation check](../img/nvm-install-check.PNG)

</details>

## **Git installation**
#### **Why ?**
Obviously, we need the version control system `git` to use and manage git repositories.
#### **How ?**

### Ubuntu / Debian

In order to have the latest stable upstream Git version for Ubuntu & Debian we'll use the PPA provided by the download page.

- Add the PPA:
  ```shell
  $ sudo add-apt-repository ppa:git-core/ppa
  ```

- Update your packages index before installing Git:
  ```shell
  $ sudo apt update
  $ sudo apt install git
  ```

- For other distributions please look at the [git download page](https://git-scm.com/download/linux) and refer to the one you use.

You can run `git -v` to check if the software is installed

<details>
<summary>It should looks like this:</summary>

![git installation check](../img/git-install-check.PNG)

</details>

## **Docker & compose plugin Installation**
#### **Why ?**

> Container tools, including Docker, provide an image-based deployment model. This makes it **easy to share an application**, or set of services, with all of **their dependencies** across **multiple environments**. Docker also **automates deploying the application** (or combined sets of processes that make up an app) inside this container environment. 
>
>_Source: [what is docker ? - redhat](https://www.redhat.com/en/topics/containers/what-is-docker#:~:text=How%20does%20Docker%20work%3F)_

#### **How ?**

Folling the [documentation of Docker](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) we will set up the official repository before installing.

1. Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:
    ```shell
    $ sudo apt update
    $ sudo apt install ca-certificates gnupg
    ```
2. Add Docker’s official GPG key:
    ```shell
    $ sudo install -m 0755 -d /etc/apt/keyrings
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    $ sudo chmod a+r /etc/apt/keyrings/docker.gpg
    ```

3. Use the following command to set up the repository:
    ```shell
    $ echo \
      "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

- For other distributions please look at the [Docker engine download page](https://docs.docker.com/engine/install/) and refer to the one you use.

Now we can get Docker, we can update our package index and run `apt install` to install the Docker's packages.

```shell
$ sudo apt update
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

<details>
<summary>It should looks like this:</summary>

![docker and docker compose installation check](../img/docker-docker-compose-installed-check.PNG)

</details>

**Note:** Don't forget to **enable ans start** docker services before using it _obviously_.

## **Php-cli & Composer Installation (ondrej ppa)**

#### **Why ?**
We will use Composer with a plugin for **CycloneDX** in order to **create SBOM** from PHP Composer projects that we will upload to our future Dependency Track server.

**Note:** Below I have installed `php7.4`, to be able to use the latest CycloneDX plugin you need a PHP version > 8.1

#### **How ?**

The ondrej ppa contains all and also the **latest build packages** of PHP.

1. Add PHP PPA repository and update our package index and add the PPA:
    ```shell
    $ sudo add-apt-repository ppa:ondrej/php
    $ sudo apt update
    ```
2. Install the chosen version of PHP (7.4 here) and PHP modules needed:
    ```shell
    $ sudo apt install php7.4 php7.4-cli php7.4-common php7.4-curl php7.4-mbstring php7.4-mysql php7.4-xml
    ```
- For other distributions search for **"php-cli install \<your distro>"** plus the PHP modules and follow guides from **sources you can trust**.

Composer have an **convenience script** for command-line installation that we can use to install it.

```shell
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"
```

> This installer script will simply check some `php.ini` settings, warn you if they are set incorrectly, and then download the latest `composer.phar` in the current directory. The 4 lines above will, in order:
>- Download the installer to the current directory
>- Verify the installer SHA-384, which you can also cross-check here
>- Run the installer
>- Remove the installer
>
>Most likely, you want to put the `composer.phar` into a directory on your PATH, so you can simply call `composer` from any directory (Global install), using for example:
>```shell
>$ sudo mv composer.phar /usr/local/bin/composer
>```
>
>_Source: [Command-line installation - getcomposer](https://getcomposer.org/download)_

<details>
<summary>It should looks like this:</summary>

![composer installation check](../img/composer-installation.PNG)

</details>

We have installed PHP and Composer it's time to install what we will really use to create SBOM for PHP projects.

Before make sure you have `unzip` installed.

<details><summary>If not here is how</summary>

## Ubuntu / Debian

Use `apt install` to install the `unzip` package from your distro's repositories.
```shell
$ sudo apt install unzip
```
## Arch / Arch based

Use `pacman -S` to install the `unzip` package from your distro's repositories.
```shell
$ sudo pacman -S unzip
```

For other distributions I'm pretty sure if you try to install `zip` or `unzip` with your default package manager it will be available and possibly already installed by default.

</details>

On the [CycloneDX PHP Composer Plugin repository](https://github.com/CycloneDX/cyclonedx-php-composer) we found two different types of installation.
1. As a **global** Composer plugin:
    ```shell
    $ composer global require cyclonedx/cyclonedx-php-composer
    ```
2. As a **developement dependency** of your current project:
    ```shell
    $ composer require --dev cyclonedx/cyclonedx-php-composer
    ```

Check that CycloneDX Plugin is installed by running `composer` and looking whether the last line is CycloneDX or not.

It should look like this:

<details>
<summary>It should looks like this:</summary>

![cyclonedx php composer plugin installed](../img/cyclonedx-installed-php-composer.PNG)

</details>

---

# **Install & Start Dependency Track**
Now that all we can need have been installed it's time to download, start and do our first Component Analysis.

## Download & Install Dependency Track
---
Let's make and enter a new directory for the Dependency Track Docker compose file we're about to download.

```shell
$ mkdir dependency-track
$ cd dependency-track
```

Following the [Dependency Track official documentation](https://docs.dependencytrack.org/getting-started/deploy-docker/) we can download the latest version of Dependency Track by running:

```shell
$ curl -LO https://dependencytrack.org/docker-compose.yml
```

All we have to do now is run our brand new `docker-compose.yml` file with `docker compose`.

To create and start the Dependency Track server:
```shell
$ sudo docker compose up -d
```
<details>
<summary>It should looks like this:</summary>

![dependency track docker installation](../img/dependency-track-docker-compose.PNG)

![dependency track server started](../img/dependency-track-started.PNG)

</details>

# First Launch
Our docker compose command ended successfully but how do we see the frontend and api server ? How to we make and upload SBOM ?

Let's get into this.

```shell
$ sudo docker ps -a
```
This command outputs all the containers and informations about them.

<details>
<summary>It should looks like this:</summary>

![dependency track docker containers](../img/docker-ps--a.PNG)

</details>

We can look at the `PORTS` section and see that the frontend and api server are running respectively on `0.0.0.0:8080` and `0.0.0.0:8081`.

In your browser at `localhost` on port `8080` you'll find the Dependency Track server plateform.

The **default credentials** of the Dependency Track plateform **can be found [here](https://docs.dependencytrack.org/getting-started/initial-startup/)**.

After your **first login** for not **obvious** security reasons you'll be asked to **change the default password**.

# Usage Exemple
For example I'll use one of my old dev repositories that used to be a good discord bot written in `JavaScript` and running with `Node.js`, now it's more a good exemple for Dependency Track.

Let's clone my repo in a projects directory.

```shell
$ mkdir projetcs
$ cd projetcs
$ git clone -b 0.0.4 https://github.com/marwank270/proto-old.git
$ cd proto-old
```

<details>
<summary>It should looks like this:</summary>

![usage exemple begining](../img/usage-env.PNG)

</details>

## **Setting up the project**
We previously installed `nvm` in our system and its time has come.

Use `nvm ls-remote` to fetch a list of all versions available.

My personal project was originally runned with **Node.js-v14.21.3(LTS: Fermium)**, some changes were made and I switched to **Node.js-v16.20.2(LTS: Gallium)**. With that said:
- I'm gonna pull the latest v16 with `nvm` with:

  ```shell
  $ nvm install v16
  ```

- Do a quick check by using `nvm list` to list all the versions currently installed on our system which one is in use by nvm and the system.

  ```shell 
  $ nvm list
  $ node -v
  ```
  <details><summary>It should looks like this:</summary>

  ![nvm node v16 install](../img/nvm-node-v16.PNG)

  ![nvm node v16 install check](../img/nvm-node-v16-install-check.PNG)

  </details>

- Install the project dependencies:

  ```shell
  $ npm install
  ```

Time to create the project on Dependency Track. In the '_Projects_' tab we have a '_Create Project_' button, and we can fill up the projects informations.

<details><summary>Open to view screenshots</summary> 

![create project](../img/create-project.PNG)

![created project](../img/created-project.PNG)

</details>

In not so long we'll need two strings that we can save now for later:

1. The project **UUID** or **Object Identifier**, we can find it in the newly created project's row, in '_View Details_', Last line of '_General_' tab:

    <details><summary>Open for screenshot</summary> 
    
    ![project details and uuid](../img/project-details-uuid.PNG)
    
    </details>

2. The API key of the server, it can be found inside '_Administration_', under '_Access Management_', in '_Teams_' (you'll see that the only member that already has an API Key is _Automation_)
  
    <details><summary>Open for screenshots</summary> 
    
    ![authomation](../img/api-key-teams-automation.PNG)
  
    ![api key copy](../img/api-key-copy.PNG)
    
  </details>
    
**Copy in a text file or come to this section when it's time to upload.**    

## **Making a Software Bill Of Material (SBOM)**
We will now use **CycloneDX** to produce the first **SBOM** of the project. 

> **SBOM** are files ***delaring every components used*** to build the project, we can compare a SBOM to the list of ingredients on food packaging.

#### **Install & Run CycloneDX with `npx`** 

```shell
$ npx @cyclonedx/cyclonedx-npm --output-file sbom.json --ignore-npm-errors
```

This line will first fetch then pull the lastest version of `cyclonedx-npm` package and finally execute it using the `--output-file` option, with the filename I want as the argument, and `--ignore-npm-errors` to do what the option's name says.

<details><summary>It should looks like this:</summary>

![proto sbom exemple](../img/proto-sbom.PNG)

</details>

## **Uploading Results to Dependency Track for analysis**
In Dependency Track's documentation under Usage section there is [Continuous Integration & Delivery](https://docs.dependencytrack.org/usage/cicd/) page that gives us comme bash code to upload our CycloneDX SBOM. We will do a script out of it.

> From this base I'm gonna **start writing** later a **more functional and flexible script** (or maybe even a **full program**)
> - [ ] Update this quote section with a link to the repository of this little project when done.

```shell
$ touch upload.sh
$ vim upload.sh
```
`upload.sh`
```bash
#!/bin/bash
# Upload sbom.json file to Dependency Track Server using some API-key and project UUID

cat > payload.json <<__HERE__
{
  "project": "dependency-track-project-uuid",
  "bom": "$(cat sbom.json | base64)"
}
__HERE__

curl -X "PUT" "http://localhost:8081/api/v1/bom" \    # Use here the API server port
     -H 'Content-Type: application/json' \
     -H 'X-API-Key: api-key' \
     -d @payload.json
```
What this script will do is output the content of your `sbom.json` file encoded in `base64` for the convenience of Dependency Track in a new `payload.json` file, then it will upload the result to the API server with `curl`.

Variables content:

- `api-key` ->  API key you now know where it is
- `dependency-track-project-uuid` -> Project UUID, you know where to find 

Now if you execute this script inside a project as I did, you will see that few minutes later (dependending on the project's size obviously) you should see on your server the results of the analysis.

![upload token](../img/upload-token.PNG)
![dependency track analysis](../img/dependency-track-final.PNG)

And results have been submitted to Dependency Track successfully !