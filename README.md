# demo_fastApi
fastapi - the framework we're using to build this backend
uvicorn - the ASGI server we'll use to serve up our app
pydantic - validation library baked into fastapi that we'll use to handle data models at different stages throughout our application
email-validator - allows pydantic to validate emails
create a server
Ok. Finally time for a little python. Only a little though.

Create a file called server.py inside the api directory, along with an __init__.py file.

touch backend/app/api/__init__.py backend/app/api/server.py
Inside server.py, add the following:

server.py
from fastapi import FastAPI
from starlette.middleware.cors import CORSMiddleware
def get_application():
    app = FastAPI(title="Phresh", version="1.0.0")
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
    return app
app = get_application()
A few interesting things going on here. We have a factory function that returns a FastAPI app with cors middleware configured. Don't worry too much about the cors stuff - this is a rabbit hole that I don't feel like diving into at the moment. If you want to read more, MDN has some great docs on it.

You'll also notice we're importing this middleware from the starlette package. FastAPI is built on top of starlette, and we'll occasionally dip into the underlying architecture to accomplish a few things. Just a heads up. You don't need to worry too much about this either, but feel free to checkout the docs here if you want to learn more.

The developers behind FastAPI have been hard at work trying to create an interface over most of the Starlette architecture, so it's actually possible to import this directly from fastapi now.

server.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
def get_application():
    app = FastAPI(title="Phresh", version="1.0.0")
    app.add_middleware(
        CORSMiddleware,
        allow_origins=["*"],
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
    return app
app = get_application()
spin up a docker container
That was fun. Now let's do a little docker handywork.

In your Dockerfile, add the following code. No need to understand this all yet either - we'll dive into the specifics later on.

Dockerfile
FROM python:3.8-slim-buster
WORKDIR /backend
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONBUFFERED 1
# install system dependencies
RUN apt-get update \
  && apt-get -y install netcat gcc postgresql \
  && apt-get clean
# install python dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt /backend/requirements.txt
RUN pip install -r requirements.txt
COPY . /backend
We pull the slim-buster docker image for python 3.8, set the working directory, and add environment variables to prevent python from writing pyc files to disc and from buffering stdout and stderr.

Then, we copy over the requirements.txt file, install the necessary dependencies, and copy our app into the backend folder.

Like I said - not a big deal if you're not up to speed with many of the commands here. It's not our focus, though we will dissect it in more detail in a future post.

If you do want more info, the majority of this docker setup is derived from Michael Herman's excellent TestDriven.io tutorial. A link to his full course on FastAPI can be found at the bottom of this post.

In your docker-compose.yml file, add the following:

docker-compose.yml
version: '3.8'
services:
  server:
    build:
      context: ./backend
      dockerfile: Dockerfile
    volumes:
      - ./backend/:/backend/
    command: uvicorn app.api.server:app --reload --workers 1 --host 0.0.0.0 --port 8000
    env_file:
      - ./backend/.env
    ports:
      - 8000:8000
A few things going on here

We're setting up our first service - server - and telling it to build using the Dockerfile we just defined.
We're saving the backend files to volume. More on this later.
We'll serve up our application with uvicorn, and host the backend on localhost:8000.
All other environment variables will be taken from our .env file.
And finally - lets build our docker container and get our server up and running.

bootstrapping our application
To build the appropriate Docker container, run the following from your terminal:

docker-compose up -d --build
This will take a little while, so sit back and sip on that coffee you made earlier.

When it's done building, enter your container with:

docker-compose up

FastAPI documentation
Starlette documentation
Docker documentation
TestDriven.io course - Test-Driven Development with FastAPI and Docker
Toptal: High-performing Apps with Python – introductory blog post focused on building a todo app from scratch with FastAPI using the SQLAlchemy ORM.
