# Node.js Auto Github Action Deployment

## 1. Generate ssh key
```bash
ssh-keygen -t rsa -b 4096
```
```bash
cat /root/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
```bash
cat /root/.ssh/id_rsa
```
Copy the private key to clipboard.
## 2. Create Github Action & Secrets.
### Go to Github repository Page > Settings > Secrets and Variables then create a few secrets key.
- SSH_PRIVATE_KEY = The private key copied earlier.
- SSH_HOST = The ip address of the vps.
- SSH_USER = The vps user (ex. root)
- WORK_DIR = The current working directory (optional)
- BRANCH = The repository branch (ex. main) (optional)
- APP_NAME = The project folder name that contain nodejs code.
- GH_USERNAME = The Github Username.
- GH_REPO_TOKEN = The Github token that give permission to private repository.
### Now go to Action tab and create a .yml (ex. main.yml) file. inside .yml file should be look like the code below.
```yml
on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy_and_restart:
    name: Deploy repo and restart server
    runs-on: ubuntu-latest

    steps:
      - name: Install SSH key
        run: |
          install -m 600 -D /dev/null ~/.ssh/id_rsa
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.SSH_HOST }} > ~/.ssh/known_hosts

      - name: Deploy with read-only token and restart
        env:
          BRANCH: ${{ secrets.BRANCH }}
          SSH_USER: ${{ secrets.SSH_USER }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
          REPO_URL: "https://${{ secrets.GH_USERNAME }}:${{ secrets.GH_REPO_TOKEN }}@github.com/${{ secrets.GH_USERNAME }}/${{ secrets.APP_NAME }}.git"
          WORK_DIR: ${{ secrets.WORK_DIR }}
          APP_NAME: ${{ secrets.APP_NAME }}
        run: |
          ssh ${SSH_USER}@${SSH_HOST} "cd ${WORK_DIR} && rm -rf ${APP_NAME} && git clone --branch ${BRANCH} ${REPO_URL} ${APP_NAME} && cd ${APP_NAME} && export NVM_DIR=~/.nvm && source ~/.nvm/nvm.sh && npm install && npm run start"

      - name: Clean up SSH
        run: rm -rf ~/.ssh
```
