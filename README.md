# MAAP Jupyter Interface

This repo includes JupyterLab extensions that have been built for the MAAP project (https://che-k8s.maap.xyz/)

In order to use each of these extensions they must be installed and enabled. Instructions for each extension can be found in the respective folder. 

These extensions have been developed on `Jupyter 4.4.0` and `Jupyter Lab 0.32.1`.

To build additional extensions for the project, it is recommended to start from a [cookie-cutter](https://github.com/jupyterlab/extension-cookiecutter-ts) or off a previously built extension.

Some Jupyter Extensions/Resources we have found helpful:
* [xkcd tutorial](https://jupyterlab.readthedocs.io/en/stable/developer/xkcd_extension_tutorial.html) Great place to start for extending JupyterLab
* [pizzabutton](https://github.com/peterskipper/pizzabutton) Implements a basic server extension and connects it with UI
* [cookie-cutter-ts](https://github.com/jupyterlab/extension-cookiecutter-ts) Base to build UI extensions off of
* [jupyterlab_autoversion](https://github.com/timkpaine/jupyterlab_autoversion)
* [jupyterlab-logout](https://github.com/zgqallen/jupyterlab-logout)
* [jupyterlab_iframe](https://github.com/timkpaine/jupyterlab_iframe)
* [jupyterlab_toastify](https://github.com/fcollonval/jupyterlab_toastify)
* [jupyter-notify](https://github.com/ShopRunner/jupyter-notify)
* [jupyterlab](https://github.com/jupyterlab/jupyterlab) A lot of figuring out how to add things where has happened through looking at the source code of jupyter
* [Jupyterlab-html](https://github.com/mflevine/jupyterlab_html) 
* [Jupyterlab-sandbox](https://github.com/canavandl/jupyterlab_sandbox)

##### (The following documentation should probably move somewhere else but I am going to leave it here for now)
Our development proccess involves building and running an extension locally in a conda env before installing it on the che server. Then packaging it into a docker image with the rest of the extensions. At the point of incorperating the extension into the Docker image, some minor changes may have to be made (mainly path issues).

Once on the che server:
- Under `~/che/dockerfiles/jupyterlab-extensions` install your extension.
    - You can create a separate stack to test the extension on che first before integrating with the others
- Attempt to build both server and lab extension locally on this machine. Install any of the packages it is yelling about.  Here are some common ones:
    ```bash
    npm install typescript
    npm install tsc

    npm install @jupyterlab/application
    npm install @jupyterlab/launcher
    npm install @phosphor/widgets
    npm install @jupyterlab/apputils
    npm install @jupyterlab/coreutils
    npm install @types/node --save
    npm install @types/jquery --save
    ```
- Add your install to the Dockerfile. For example:
    ```bash
    # jlab pull projects into /projects directory
    RUN cd /pull_projects && npm run build
    RUN cd /pull_projects && jupyter labextension link .
    RUN cd /pull_projects && pip install -e .
    RUN cd /pull_projects && jupyter serverextension enable --py pull_projects --sys-prefix
    
    ```
- If your extension includes a server extension you also need to modify `entrypoint.sh`. This is because jupyter server extensions function off of having a standard base url, but in the context of che the url is not what jupyter thinks it is.
- Here is some magic that fixes it (add this line and replace with the path to where your `load_jupyter_server_extension` function is)
    ```bash
    perl -pi -e "s|web_app.settings\['base_url'\]|'/'|g" /show_ssh_info/show_ssh_info/__init__.py
    ```
- Then rebuild the docker image. `microk8s.docker build -t localhost:32000/che-jupyterlab-extensions .`
- Push! `microk8s.docker push localhost:32000/che-jupyterlab-extensions `
- Now when you build a new workspace with the `localhost:32000/che-jupyterlab-extensions` image it will automatically fetch the new image. (found in the stack's `Recipe` or `Raw Configuration`)
    - you can also specify the image tag to use in your build on the stack if you want to use a previous build