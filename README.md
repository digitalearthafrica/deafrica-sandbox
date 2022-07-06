# DEA Sandbox

[![Test and Push](https://github.com/digitalearthafrica/deafrica-sandbox/actions/workflows/docker-sandbox-cache.yml/badge.svg)](https://github.com/digitalearthafrica/deafrica-sandbox/actions/workflows/docker-sandbox-cache.yml)
![Vulnerability Scan](https://github.com/digitalearthafrica/dea-sandbox/workflows/Vulnerability%20Scan/badge.svg)

Digital Earth Africa Sandbox Docker build, configuration and planning.

Please ensure all tasks have a description that includes a clear definition of when the task is complete.

[Kanban](https://github.com/digitalearthafrica/dea-sandbox/projects/1)

[Issues](https://github.com/digitalearthafrica/dea-sandbox/issues)

## Automated builds

Builds are run automatically out of this repository. Basic workflow is:

- Create a branch and implement your changes
- The Docker image is built on the branch and as part of the pull request
- Once the PR is merged, a build will be run and pushed to `latest` and `sudo-latest` from master
- When a new stable image is ready, create a release with a version number like `0.0.6` and
this will be pushed to `0.0.6` and `sudo-0.0.6`.

## Local environment

A simple local environment that can be used to test the JupyterHub system in can be started using Docker Compose
with the command `docker-compose up` and browsing to http://localhost:8888 and adding the token that is displayed
on your terminal after starting the system.

To run connected to a database in one of the DEA systems, you'll need to start a Kubernetes port forwading process
with a command like `port-forward -n service deployment/pg-proxy 5432:5432`.

And then set up a file in the root of this folder `.docker.env` with connection details in it. Use the
`.docker.env.example` as a template for this file. You then want to run the Docker Compose environment without a
postgres database, so use the command `docker-compose -f docker-compose.yml up` to start it. This will ignore
the `docker-compose.override.yml` file, which provides a postgres container.

Any files in the `./notebooks` folder will be mounted in the user's home folder. That is to say that `./notebooks`
will be mounted at `/home/jovyan`/


## Testing the digitalearthafrica/deafrica-sandbox-notebooks

To test the DE Africa Analysis Sandbox notebooks in the [DEV sandbox](https://sandbox.dev.digitalearth.africa/), follow the instructions below:

- In the [DEV sandbox](https://sandbox.dev.digitalearth.africa/) open a  new terminal.
- Install the packages `nbmake` , `pytest-xdist` and `pytest-reportlog`:
    
    ```
    python -m pip install --upgrade nbmake pytest-reportlog pytest-xdist
    ```

- Check if everything is set up correctly by instructing `pytest` to simply collect (but not run) all notebooks:

    ```
    pytest --collect-only --nbmake -n=auto
    ```
    
- Now that you have validated that `nbmake` and `pytest` are working together and that they can see all the notebooks, run the test for real and save the report: 

    ```
    pytest --nbmake -n=auto --report-log="pytest_results.txt"
    ```


### Document warnings in the notebooks

To find the [warnings](https://docs.python.org/3/library/warnings.html#warning-categories) like `FutureWarnings` or `UserWarning` occuring in the notebooks, you will need to run the individual notebooks and document the warnings in an Excel or Google sheet for future reference. 
