frrbot
======

A GitHub bot for managing the FRRouting/frr repo.

Setup
-----
1. Install Python 3
2. Clone repo & `cd frrbot`
3. Install [poetry](https://python-poetry.org/docs/#osx-linux-bashonwindows-install-instructions)
4. `poetry install`
5. Create and configure a GitHub App on GitHub, save the webhook secret, APP ID
   and private key
6. Copy `config.yaml.example` to `config.yaml` and fill in the fields appropriately

Running
-------

Notes:

* Because this program invokes git from the shell, it needs to set the
  `user.name` and `user.email` options. Presently it does this globally so it
  will overwrite your configs for these. Running in Docker is recommended until
  this is fixed.

* When setting `gh_app_route`, remember that this is relative to the uWSGI
  mountpoint. So for example if your uWSGI mountpoint is `/frrbot` and you set
  `gp_app_route` to `/gh`, the full URL that frrbot will listen for GitHub
  webhooks on will be `/frrbot/gh`.

**Option 1: `flask run`**

1. Set environment variable `FLASK_APP=frrbot.py`
2. Fill out config file
3. Execute `flask run`
4. Configure your web server of choice to proxy your payload URL to
   `http://localhost:5000/` and reload it

**Option 2: WSGI**

1. Install [uwsgi](https://uwsgi-docs.readthedocs.io/en/latest/)
2. Fill out config file
3. Use `./run.sh` to create and mount a WSGI endpoint on `/frrbot` and
   configure your web server to WSGI proxy your payload URL to it

**Option 3: Docker**

1. `docker build --tag frrbot:latest .`
2. `docker run -e GH_WEBHOOK_SECRET=<secret> GH_APP_ROUTE="/gh" GH_APP_ID=<ID> -e GH_APP_PKEY_PEM_PATH=<path> -e JOB_STORE_PATH=<path> --port 80:80 frrbot:latest`
3. frrbot will be listening on `0.0.0.0:80/frrbot/gh`


Notes for docker:

* Environment variables are the preferred way to configure the container but
  it's also possible to mount `config.yaml` into the path `/app/config.yaml` if
  desired
* `GH_WEBHOOK_SECRET` should be the webhook secret used to authenticate requests from GitHub
* `GH_APP_ID` should be the ID of your GitHub App
* `GP_APP_ROUTE` should be the URL you want to listen for webhooks on
  *relative to the uWSGI mountpoint*
* `GH_APP_PKEY_PEM_PATH` should be the absolute path to the PEM format private
  key associated with your GitHub App; you should mount the key into the
  container at that path
* `GH_GIST_USER_TOKEN` should be a personal access token for a real user. This
  user will be used to host gists, since GitHub apps can't use gists.
* Since the job store should outlive the container, you should mount a volume
  where the job store should live from the host into the container before
  running and set `JOB_STORE_PATH` appropriately. For example:

  ```docker run --mount type=bind,source=/opt/frrbot,target=/frrbot ... -e JOB_STORE_PATH=/frrbot/jobstore.sqlite frrbot:latest```

* uWSGI options within the container can be modified by changing `uwsgi.ini` in
  this repository root and rebuilding the container.
