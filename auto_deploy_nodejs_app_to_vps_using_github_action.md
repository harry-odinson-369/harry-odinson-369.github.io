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
## 2. Create Github Action secrets.
### Go to Github repository Page > Settings > Secrets and Variables then create a few secrets key.
- SSH_PRIVATE_KEY = The private key copied earlier.
- SSH_HOST = The ip address of the vps.
- SSH_USER = The vps user (ex. root)
- WORK_DIR = The current working directory (optional)
- BRANCH = The repository branch (ex. main) (optional)

