# DEPLOY FLASK WITH MARIADB WITH DOCKER-COMPOSE

First let's install docker-compose. Connect to your raspberry PI then issue the following commands

```
sudo apt install docker-compose
```

Make sure your stop and remove any running container, especially the db ones if you have any.

We begin with the following project layout:

```
dockerize/
├── app
│   └── app.py
└── db
    └── init.sql
```

- *app.py* — contains the Flask app which connects to the database and exposes one REST API endpoint
- *init.sql* — an SQL script to initialize the database before the first time the app runs

Start creating the folders then the `app.py` file

```
cd ~
mkdir dockerize
cd dockerize
mkdir db
mkdir app
cd app
nano app.py
```

Cut and paste the following code, then save and exit.

```
from typing import List, Dict
from flask import Flask
import mysql.connector
import json

app = Flask(__name__)


def test_table() -> List[Dict]:
    config = {
        'user': 'root',
        'password': 'root',
        'host': 'db',
        'port': '3306',
        'database': 'devopsroles'
    }
    connection = mysql.connector.connect(**config)
    cursor = connection.cursor()
    cursor.execute('SELECT * FROM test_table')
    results = [{name: color} for (name, color) in cursor]
    cursor.close()
    connection.close()

    return results


@app.route('/')
def index() -> str:
    return json.dumps({'test_table': test_table()})


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

then create the `init.sql` file

```
cd ~/dockerize/db
nano init.sql
```

Cut and paste the following code, then save and exit. 

```
create database devopsroles;
use devopsroles;

CREATE TABLE test_table (
  name VARCHAR(20),
  color VARCHAR(10)
);

INSERT INTO test_table
  (name, color)
VALUES
  ('dev', 'blue'),
  ('pro', 'yellow');
```

## Creating a Docker image for our app 

We want to create a Docker image for our app, so we need to create a Dockerfile in the app directory. A Dockerfile contains a set of instructions describing our desired image and allow its automatic build.

```
cd ~/dockerize/app
nano Dockerfile
```

Cut and paste the following code, then save and exit. 

```
# Use an official Python runtime as an image
FROM python:3.8

# The EXPOSE instruction indicates the ports on which a container listens to
EXPOSE 5000

# Sets the working directory for following COPY and CMD instructions
# Notice we haven’t created a directory by this name - this instruction
# creates a directory with this name if it doesn’t exist
WORKDIR /app

COPY requirements.txt /app
RUN python3 -m pip install --upgrade pip
RUN pip3 install --trusted-host pypi.org --trusted-host pypi.python.org --trusted-host=files.pythonhosted.org --no-cache-dir -r requirements.txt

# Run app.py when the container launches
COPY app.py /app
CMD python3 app.py
```

What this does is simply as described in the file — base the image on a Python 3.8 image, expose port 5000 (for Flask), create a working directory to which requirements.txt and app.py will be copied, install the needed packages and run the app.

We need our dependencies (Flask and mysql-connector) to be installed and delivered with the image, so we need to create the aforementioned requirements.txt file:

```
cd ~/dockerize/app
nano requirements.txt
```

Cut and paste the following code, then save and exit. 

```
Flask
mysql-connector
```

Now we can create a docker image for our app, but we still can’t use it, since it depends on MySQL, which, as good practice commens, will reside in a different container. We will use docker-compose to facilitate the orchestration of the two independant containers into one working app.

## Creating a docker-compose.yml

So let’s create a new file, `docker-compose.yml,` in our project’s root directory:

```
cd ~/dockerize
nano docker-compose.yml
```

Cut and paste the following code, then save and exit. 

```
version: "3"
services:
  app:
    build: ./app
    links:
      - db
    ports:
      - "5000:5000"
  db:
    image: lscr.io/linuxserver/mariadb
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - MYSQL_ROOT_PASSWORD=root
      - TZ=Europe/London
      - MYSQL_USER=root
      - MYSQL_PASSWORD=root

    volumes:
      - ./db:/docker-entrypoint-initdb.d/:ro
    ports:
      - 32000:3306
    restart: unless-stopped
    
```

We are using two services, one is a container which exposes the REST API (app), and one contains the database (db).

- *build:* specifies the directory which contains the Dockerfile containing the instructions for building this service
- *links:* links this service to another container. This will also allow us to use the name of the service instead of having to find the ip of the database container, and express a dependency which will determine the order of start up of the container
- *ports:* mapping of <Host>:<Container> ports.*image:* Like the FROM instruction from the Dockerfile. Instead of writing a new Dockerfile, we are using an existing image from a repository. It’s important to specify the version — if your installed mysql client is not of the same version problems may occur.

- *environment:* add environment variables. The specified variable is required for this image, and as its name suggests, configures the password for the root user of MySQL in this container. More variables are specified here.
- *ports:* Since there already is a running mysql instance on my PI using the port 3306, it is mapped to a different one. 

## Running the app

In order to run the our dockerized app, issue the following command from the terminal:

```
$ cd ~\dockerize
$ sudo service docker restart
$ docker-compose up
```

You can see the image being built, the packages installed according to the `requirements.txt,` etc. If everything went right, you will see the following line:

```
app_1  |  * Running on http://0.0.0.0:5000/ 
app_1  |  * Running on http://127.0.0.1:5000
app_1  |  * Running on http://172.21.0.2:5000
(Press CTRL+C to quit)	
```

Before we check the app, let's connect to `mysql` and create the database. Open a new Terminal window and connect to your PI. Note that you will first need to install `mariadb-server` on the pi to be able to use the `mysql` command.

```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mariadb
==>172.21.0.2

mysql -h 172.21.0.2 -u root -p

create database devopsroles;
use devopsroles;

CREATE TABLE test_table (
  name VARCHAR(20),
  color VARCHAR(10)
);

INSERT INTO test_table
  (name, color)
VALUES
  ('dev', 'blue'),
  ('pro', 'yellow');

exit;

```

on your laptop start a browser and connect to the IP of your PI on port 5000.

You should see the following output.

```
{"test_table": [{"dev": "blue"}, {"pro": "yellow"}]}
```

