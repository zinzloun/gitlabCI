# GitLab CE DevSecOps CI
This is repo provides setup instruction to create a Gitlab CE DevSecOps CI pipe
## Architecture
This is a lab enviroment and I keep it simple: I installed Gitlab and the runner on the same machine, that it's not a appropriate configuratuion for a producion enviroment.
## Reuirements
### Hardware
- 2 vCPU
- 8GB RAM
- 50GB HDD
### OS
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
### Software
- GitLab CE. You can follows this guide: https://wiki.crowncloud.net/?How_to_Install_Gitlab_On_Debian_12. Since this is a lab, I didn't configure HTTPS, of course it would be a necessary step on a production enviroment. So following the guide, set the external_url parameter to:

      external_url 'http://<your hostname>'
  make sure you can resolve the name with your local IP (not 127.0.0.1), in case adjust your hosts file
- Gitlab runner
