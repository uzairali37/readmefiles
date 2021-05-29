
<!-- PROJECT LOGO -->
<p align="center">
  
  <!-- ![image](images/logo1.png) -->
 <img src="./images/logo1.png" width="50%" alt="OpenStack">
</p>

<br />

<p align="center">
  <a href="https://github.com/SuperboGiuseppe/linux-on-demand">
    <img src="./images/graph_terraform.png" width="50%" alt="OpenStack">
    <!-- <img src="https://easybase.io/assets/images/logo_black.png" alt="easybase logo black" width="80" height="80"> -->
  </a>
</p>

<br />

<p align="center">
  <img alt="npm" src="https://img.shields.io/npm/dw/easybase-react">
  <img alt="GitHub" src="https://img.shields.io/github/license/easybase/easybase-react">
  <img alt="npm bundle size" src="https://img.shields.io/bundlephobia/min/easybase-react">
  <img alt="npm" src="https://img.shields.io/npm/v/easybase-react">
</p>

<br />

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [About the Project](#about-the-project)
  * [Architecture](#Architecture)
  * [Built With](#built-with)
* [Getting Started](#getting-started)
  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
* [Usage](#usage)
* [Documentation](https://easybase.io/docs/easybase-react/)
* [Examples](#examples)
* [Troubleshoot](#troubleshoot)
* [License](#license)
* [Contact](#contact)



<!-- ABOUT THE PROJECT -->
## About The Project
In this project, we are going to design and configure an infrastructure that hosts several development web applications, especially for computer
science/engineering students. In order to configure the necessary infrastructure assets, we are going to use OpenStack as our IaaS
platform. In addition to this, we are going to adopt Docker and Kubernetes PaaS solutions to deploy the required containerized web
applications.

### Architecture
The main OpenStack service used for this infrastructure is **Nova**,
which is used for the server instances. These instances are based on
different OS images, depending on their main functionality. In addition
to this, additional necessary applications/packages will be installed in
each instance.

**Infrastructure Design** as you can see in the image given below.

![image](images/Infrastructure_Architecture.png)

Servers security is based on the ”Security Group” concept, which is
similar to Firewall rule set but is applied on a group of servers level.
Both the inbound and outbound traffic are filtered by the Security
Group. Only defined ports and IPs are enabled to incoming traffic, as
well as defined ports are enabled in outbound connections. Traffic is
permitted on a need bases.

**Networking**
The base OpenStack service that creates the structure of the network
subnets that acts like a container for the instances is **Neutron**. It
is possible to create a dedicated network for the instances of the
project. In this case the network is labeled ”edu-private-network-01”
and any server created will be placed in this network. Different subnets
can be allocated in the network. For this project a single private
subnet labeled “edu-private-network-subnet-01” is allocated.

![image](images/Network_Design.png)

| Subnet name                   | Network subnet | Public/Private | Gateway IP |
|-------------------------------|----------------|----------------|------------|
| lod-private-network-subnet-01 | 10.0.2.0/24    | Private        | 10.0.1.1   |


Private subnet is restricted only to be accessible through the dedicated
router interface between private project network and public network. The
router named ”edu-router-01” is connected to the private subnet through
an interface. In this way it is possible to reach public internet from
the subnet and viceversa when allocating a floating IP.

| Router name   | Availability zone | Interface                                 | Gateway IP |
|---------------|-------------------|-------------------------------------------|------------|
| lod-router-01 | Nova              | Public <--> lod-private-network-subnet-01 | 10.0.1.1   |

**Asset inventory**
This section will describe the servers from the hardware and software
point of view. The table below summarises the hardware resources of the
servers in place for the project.

| Host      | Flavor    | Number of vCpus | RAM | Storage | OS Image     | Floating IP |
|-----------|-----------|-----------------|-----|---------|--------------|-------------|
| lod-bh-01 | m1.small  | 1               | 2GB | 20GB    | Ubuntu 18.04 | Yes         |
| lod-fe-01 | m1.small  | 1               | 2GB | 20GB    | Ubuntu 18.04 | Yes         |
| lod-be-01 | m1.medium | 2               | 4GB | 40GB    | Ubuntu 18.04 | No          |
| lod-db-01 | m1.small  | 1               | 2GB | 20GB    | Ubuntu 18.04 | No          |

**Bastion Host (edu-bh-01)**
This instance has the purpose of providing access to a private network
from the public network. Its scope is to minimize the chance of
penetration and so strict firewall rules are required to be applied. For
this reason a floating IP is associated to this instance and the only
port reachable from public network is 22, as only SSH connection will be
possible from the floating IP. Naturally only the known hosts will be
authorized to access the private network through the exchange of public
and private keys. From the bastion host it will be possible to reach the
other project instances from their private IP addresses.

**Front-end (edu-fe-01)**
The front-end instance is in charge of hosting the WEB Interface for the
final user. For this reason **Nginx** will be installed during the
creation of the instance in order to configure a web server. In order to
expose the web server content to the public internet it is necessary to
associate a floating IP. The content will be reachable only through
HTTP/HTTPS protocol (Ports 80 and 443). All the services prompted by the
front-end are hosted in the back-end, and the user can access them only
if it is registered to the platform with a private profile.

**Back-end (edu-be-01)**
The back-end instance is the core of the project as all the main web
applications are executed here. In order to make educational
applications available, docker PaaS set of service will be installed
along with the docker engine. In this way we will deploy the following
containers:

| Application | Description                                             | Link                                                                       |
|-------------|---------------------------------------------------------|----------------------------------------------------------------------------|
| SSHwifty    | SSHwifty is a SSH and Telnet connector made for the web | [SSHwifty repository](https://github.com/nirui/sshwifty)                   |
| Paperless   | PDF documents manager                                   | [Paperless repository](https://github.com/the-paperless-project/paperless) |
| Code-Server | Visual studio code IDE on WEB                           | [Code-Server repository](https://github.com/cdr/code-server)               |

In addition to these docker container applications, kubernetes
environment will be installed and configured in this instance as it is
necessary to manage a dynamic cluster of pods. For this reason a
kubernetes deployment will be configured along its replication set of
pods. These pods are going to be offered to the users as Linux
environment sandbox. For this reason the user can create and access a
pod whenever he needs to learn some Linux bash commands. Thanks to
SSHwifty the user will be able to establish an SSH connection with the
private pod directly via browser. All the pods will be based on standard
Ubuntu image.

User profiles management will be managed by the back-end through the
3306 port from which it is possible to communicate with the
infrastructure database (edu-db-01).

**Database (edu-db-01)**
In the database we will have all the details of user profiles. The user
can access the services only if authorized by his credentials, which are
stored in this database. For this reason mysql-server package will be
installed in order to store a relational database in the root volume of
this instance.

### Built With

* [Terraform](https://www.terraform.io/)
* [OpenStack](https://www.openstack.org/)
* [kubernetes](https://kubernetes.io/)
* [Docker](https://www.docker.com/)


<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

* pendding
* pendding

### Installation

```sh
bla bla bla
```

### Database
#### Create of Project

<p align="center">
  <img src="./assets/react-integration-3.gif" width="80%" alt="react easybase integration 1">
  <br />
  <br />
  <img src="./assets/users-2.gif" width="80%" alt="react easybase integration 2">
</p>

#### Then, download your token and place it at the root of your project

<pre>
├── src/
│   ├── App.js
│   ├── index.js
│   └── ebconfig.js
├── assets/
├── package.json
└── ...
</pre>

### Cloud Functions

#### Deploy a new cloud function

<p align="center">
  <img src="./assets/deploy-function-1.png" width="80%" alt="react easybase deploy function 1">
</p>

#### Take note of the automatically generated route

<p align="center">
  <img src="./assets/deploy-function-14.png" width="80%" alt="react easybase deploy function 1">
</p>

<!-- USAGE EXAMPLES -->
## Usage

### Database

Wrap your root component in *EasybaseProvider* with your credentials.
```jsx
import React, { useEffect } from "react";
import { EasybaseProvider, useEasybase } from 'easybase-react';
import ebconfig from "./ebconfig.json";

function App() {
  return (
    <EasybaseProvider ebconfig={ebconfig}>
      <Container />
    </EasybaseProvider>
  );
}
```

<br />

<details>
<summary>If you're using a project, implement a sign-in/sign-up workflow for users.</summary>
<p>

```jsx
function ProjectUser() {
  const [usernameValue, setUsernameValue] = useState("");
  const [passwordValue, setPasswordValue] = useState("");

  const {
    isUserSignedIn,
    signIn,
    signUp,
    getUserAttributes
  } = useEasybase();

  if (isUserSignedIn()) {
    return (
      <div>
        <h2>Your signed in!</h2>
        <button onClick={ _ => getUserAttributes().then(console.log) }>
          Click me only works if your authenticated!
        </button>
        <Container />
      </div>
    )
  } else {
    return (
      <div style={{ display: "flex", flexDirection: "column" }}>
        <h4>Username</h4>
        <input value={usernameValue} onChange={e => setUsernameValue(e.target.value)} />
        <h4>Password</h4>
        <input type="password" value={passwordValue} onChange={e => setPasswordValue(e.target.value)} />
        <button onClick={_ => signIn(usernameValue, passwordValue)}>
          Sign In
        </button>
        <button onClick={_ => signUp(usernameValue, passwordValue)}>
          Sign Up
        </button>
      </div>
    )
  }
}
```

</p>
</details>

<br />

<details>
<summary>Then, interface with your data in a stateful and synchronous manner.</summary>
<p>

[EasyQB](https://easybase.github.io/EasyQB/) is a powerful query builder to perform **Select**, **Update**, **Insert**, and **Delete** operations on your table. You can access this through the `.db` function. [Learn how to use the `.db` function here](https://easybase.github.io/EasyQB/). Here's an example of how to use a simple clause to read and insert data with your table.

```jsx
function Container() {
  const { db } = useEasybase();

  const [easybaseData, setEasybaseData] = useState([])
  const [currentPage, setCurrentPage] = useState(0)

  useEffect(() => {
    const mounted = async () => {
      const res = await db('MY TABLE').return().limit(10).offset(currentPage * 10).all()
      setCurrentData(res)
    }

    mounted();
  }, [currentPage]);

  const onChange = async (_key, column, newValue) => {
      await db('MY TABLE').set({ [column]: newValue }).where({ _key: _key }).one()
  }

  const onChangePage = (pageNum) => {
    setCurrentPage(pageNum)
  }

  return (
    <div>
      {easybaseData.map(ele => <Card
          {...ele}
          onChangeValue={onChangeValue}
          onChangePage={onChangePage}
        />
      )}
    </div>
  )

}
```

To see just how much functionality is packed into the `.db` function (aggregators, grouping, expressions), [check out the documentation here](https://easybase.github.io/EasyQB/).
</p>
</details>

<br />
 
Learn about using the `.db` function for [Select](https://easybase.github.io/EasyQB/docs/select_queries.html), [Update](https://easybase.github.io/EasyQB/docs/update_queries.html), [Delete](https://easybase.github.io/EasyQB/docs/delete_queries.html), or [Insert](https://easybase.github.io/EasyQB/docs/insert_queries.html) queries!

### Cloud Functions

The *EasybaseProvider* pattern is not necessary for invoking cloud functions, only *callFunction* is needed.
```jsx
import { callFunction } from 'easybase-react';

function App() {
    async function handleButtonClick() {
        const response = await callFunction("123456-YOUR-ROUTE", {
            hello: "world",
            message: "Find me in event.body"
        });

        console.log("Cloud function: " + response);
    }

    //...
}
```

<!-- DOCUMENTATION -->
## Documentation

Documentation for this library [is available here](https://easybase.io/docs/easybase-react/).

Information on **database querying** [can be found in the EasyQB docs](https://easybase.github.io/EasyQB/).

<!-- EXAMPLES -->
## Examples

[User authentication walkthrough](https://www.freecodecamp.org/news/build-react-native-app-user-authentication/)

[Deploying cloud functions](https://easybase.io/react/2021/03/09/The-Easiest-Way-To-Deploy-Cloud-Functions-for-your-React-Projects/)

<!-- TROUBLESHOOT -->
## Troubleshoot

#### React Native

For React Native users, this library will not work with [expo](https://expo.io/) due to native dependencies. Instead, use `react-native run-ios`.

Errors can arise because this library depends on [Async Storage](https://react-native-async-storage.github.io/async-storage/docs/install/) which requires *linking*. This package attempts to automatically handle this for you during postinstall (`scripts/postinstall.js`). If this script fails or you encounter an error along the lines of `Unable to resolve module '@react-native-community/async-storage'...`, here are two different methods to configure your React Native project.

  1. `npm start -- --reset-cache`
  2. Exit bundler, proceed as normal

  Or

  1. `npm install @react-native-community/async-storage@1.12.1`
  2. [Link package via these instructions](https://github.com/react-native-async-storage/async-storage/tree/9aad474ff7ca64d34ef94358a39205a609455aca#link)


#### React

No linking is required for React. If you encounter any errors during installation or runtime, simply reinstall the library.

  1. Delete `node_modules/` folder
  2. `npm install easybase-react`

<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.

<!-- CONTACT -->
## Contact

| Name           | Surname | E-mail                                    | Github repository                  |
|----------------|---------|-------------------------------------------|------------------------------------|
| Giuseppe       | Superbo | giuseppe.superbo97_at_gmail.com           | https://github.com/SuperboGiuseppe |
| Muhammad Uzair | Aslam   | muhammaduazair.aslam_at_studenti.unitn.it | https://github.com/uzairali37      |

