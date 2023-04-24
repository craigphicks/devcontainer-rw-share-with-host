

**Summary**

This explains how to enable the `devcountiner`-created container and the host both be able to write the same mounted directory.

1 - In the container configuration, the Dockerfile is modified so the group write permission flags will be set on write.

2 - On the host side, first a new group is created which matches the container group, and the host user is added to that group.  

3 - Then, the development directores and files permission flags are set. 


**1 Devcontainer config**

   Add this line to `.devcountainer/Dockerfile`:
   ```
   RUN su node -c "echo 'umask 0002' >> /home/node/.bashrc"
   ```

   For example, this a working `.devcountainer/Dockerfile`:
   ```
   ARG VARIANT=20-bullseye
   FROM mcr.microsoft.com/devcontainers/javascript-node:0-${VARIANT}
   RUN su node -c "echo 'umask 0002' >> /home/node/.bashrc"
   ```

   This is a working `devcontainer.json`
   ```
   {
      "name": "Node.js & TypeScript",
      "build": {
         "dockerfile": "Dockerfile"
      },
      "features": {
         // This git update cause a few minutes extra in rebuild, but even without it
         // we are getting version 2.40.0 which has the git vuln fixes
         // "ghcr.io/devcontainers/features/git:1": {
         //     "version": "latest",
         //     "ppa": "false"
         // }
      },

      // Configure tool-specific properties.
      "customizations": {
         // Configure properties specific to VS Code.
         "vscode": {
            // Add the IDs of extensions you want installed when the container is created.
            "extensions": [
               "dbaeumer.vscode-eslint"
            ]
         }
      },

      // Use 'forwardPorts' to make a list of ports inside the container available locally.
      // "forwardPorts": [],

      // Use 'postCreateCommand' to run commands after the container is created.
      // "postCreateCommand": "yarn install",

      // Set `remoteUser` to `root` to connect as root instead. More info: https://aka.ms/vscode-remote/containers/non-root.
      "remoteUser": "node"
   }
   ```

**2 Add group on host** 

In the case of ubuntu 20 host, the uid of files owned by the user on the container are observed to be 100999.  This will likely vary by system.

On the host create a group with gid 100999, and add the host user to that group.  It is necessary to log out and log back in again for it to take effect.

```
sudo addgroup --gid 100999 g100999
sudo usermod -a -G g100999 craig
```


**3 Set host development directory permissions** 

Ensure the whole directory and all its subdirectories have group read, write, executable, and setuid permissions. 

```
sudo find . -type d -exec chmod g+rwxs {} +
sudo find . -type f -exec chmod g+rw {} +
```

**4 Build and run the container in vscode**

```
devcontainer build .
devcontainer open .
```

**References**

[github repo: docker-rw-share-with-host](https://github.com/craigphicks/docker-rw-share-with-host)

[devcontainers provided example devcontainer.json file](https://github.com/devcontainers/images/blob/main/src/typescript-node/.devcontainer/devcontainer.json)

[devcontainers provided example Dockerfile file](https://github.com/devcontainers/images/blob/main/src/typescript-node/.devcontainer/Dockerfile)
