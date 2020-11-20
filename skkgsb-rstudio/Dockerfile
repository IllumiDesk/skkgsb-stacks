FROM jupyter/base-notebook:45bfe5a474fa AS base

## Use a tag instead of "latest" for reproducibility
FROM rocker/binder:3.6.3

## Declares build arguments
ARG NB_USER=jovyan
ARG NB_UID=1000

ENV NB_UID="${NB_UID}"
ENV NB_USER="${NB_USER}"
ENV HOME="/home/${NB_USER}"

USER root
RUN apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y \
    ca-certificates \
    fonts-liberation \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# Change the default username and update permissions
RUN usermod -l jovyan rstudio
RUN usermod -m -d /home/jovyan jovyan
RUN { grep -v "^$1:" /etc/group; echo "$1:x:$2:"; } >/etc/group.tmp \
 && mv /etc/group.tmp /etc/group

RUN chown -R "${NB_UID}" "${HOME}"

# copy files from base image
# start* includes: start-notebook.sh, start-singleuser.sh, and start.sh
COPY --from=base /usr/local/bin/start* /usr/local/bin/
COPY --from=base /usr/local/bin/fix-permissions /usr/local/bin/

# update permissions
RUN chmod a+rx /usr/local/bin/fix-permissions
RUN chmod +rx /usr/local/bin/start*

USER jovyan

WORKDIR /tmp
# update install.R to adjust R packages
COPY install.R /tmp/

# upgrade pip
RUN pip install -U pip

RUN pip install jupyterhub==1.2.1 \
    nbgitpuller==0.9.0 \
    nbresuse==0.3.* \
    psycopg2==2.8.* \
    https://github.com/IllumiDesk/nbgrader/archive/v0.64.2.zip

## Run an install.R script, if it exists.
RUN if [ -f install.R ]; then R --quiet -f install.R; fi

USER root

# install nbgrader and then disable all extensions by default
RUN jupyter nbextension install --symlink --sys-prefix --py nbgrader --overwrite \
 && jupyter nbextension disable --sys-prefix --py nbgrader \
 && jupyter serverextension disable --sys-prefix --py nbgrader

# everyone gets the nbgrader validate extension
RUN jupyter nbextension enable --sys-prefix validate_assignment/main --section=notebook \
 && jupyter serverextension enable --sys-prefix nbgrader.server_extensions.validate_assignment

# everyone gets the assignment extension
RUN jupyter serverextension enable --sys-prefix nbgrader.server_extensions.assignment_list \
 && jupyter nbextension enable --sys-prefix assignment_list/main --section=tree

ENV NB_GID=1000
# fix permissions as root
RUN fix-permissions "${HOME}" \
 && fix-permissions /usr/local/lib/R/ \
 && fix-permissions /srv/venv/

USER jovyan

WORKDIR /home/jovyan

CMD ["start-notebook.sh"]

# copy files as late as possible
USER root
COPY jupyterhub-singleuser /usr/local/bin/
RUN chmod +rx /usr/local/bin/jupyterhub-singleuser
COPY --from=base /etc/jupyter/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config_base.py
COPY jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config.py
RUN chmod a=r /etc/jupyter

USER jovyan
