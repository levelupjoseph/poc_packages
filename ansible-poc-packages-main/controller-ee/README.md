# Steps

1. $ `ansible-builder build`
1. copy ansible-controller-4.3.0.tar.gz to context/
1. copy myrequirements.yml to context/
1. Add these lines to context/Containerfile: ```
# Copy your custom collection tarball and requirements.yml file to the container
COPY ansible-controller-4.3.0.tar.gz /opt/ansible_collections/
COPY myrequirements.yml /opt/ansible_collections/requirements.yml

# Set the working directory to the directory containing the requirements.yml file
WORKDIR /opt/ansible_collections

# Install the Ansible Collection from the requirements.yml file
RUN ansible-galaxy collection install -r requirements.yml
```
1. $ `podman build context # and get container id` 
1. $ `podman login quay.io`
1. $ `podman push <CONTAINERID> quay.io/danielgoosen/controller-ee`
