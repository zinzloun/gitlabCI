# GitLab CE DevSecOps CI
      
         *.                  *.
        ***                 ***
       *****               *****
      .******             *******
      ********            ********
       ,,,,,,,,,***********,,,,,,,,,
      ,,,,,,,,,,,*********,,,,,,,,,,,
      .,,,,,,,,,,,*******,,,,,,,,,,,,
          ,,,,,,,,,*****,,,,,,,,,.
             ,,,,,,,****,,,,,,
                .,,,***,,,,
                    ,*,.
      
         _______ __  __          __
        / ____(_) /_/ /   ____ _/ /_
       / / __/ / __/ /   / __ `/ __ \
      / /_/ / / /_/ /___/ /_/ / /_/ /
      \____/_/\__/_____/\__,_/_.___/


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
- Gitlab runner: https://docs.gitlab.com/runner/install/linux-repository
- A SonarCloud account with a project configured. You will need to provide the following information to perform a scan:
  - Project Key
  - Organization Key
  - A token. Once logged in, you can generate it from: https://sonarcloud.io/account/security
- A Snyk account
  - You need an Auth token. Once logged in, you can get it from: https://app.snyk.io/account
- Docker CE installed: https://docs.docker.com/engine/install/debian/
- Python3 & Pip3: https://reintech.io/blog/install-python3-pip-debian-12
- A Github account to import a project in Gitlab

## Gitlab configuration
- Change the initial root password
- Disable sign up at <i>http://[your-hostname]/admin/application_settings/general#js-signup-settings</i>
- Create a new group
- Create a new runner for the group (Build->Runners). Set a tag, that we will use it later on the pipe configuration. For this lab I tagged the runner as <b>ssh</b>.
  -  Execute the commands indicated as sudo, accept the default values proposed and set <b>ssh</b> as excutor.
  -  Coming back to the server web gui and click View runners
  -  You should see your newly created runner as online
- Add gitlab-runner user to docker group

        sudo usermod -a -G docker gitlab-runner
        id gitlab-runner
            uid=990(gitlab-runner) gid=989(gitlab-runner) groups=989(gitlab-runner),988(docker)
## Import a project in Gitlab
- On your Github account fork the following project: https://github.com/manuelz120/extremely-vulnerable-flask-app
- On Gitlab set Github as import source: https://docs.gitlab.com/administration/settings/import_and_export_settings/#configure-allowed-import-sources
- Import the cloned repo from your Github account to Gitlab using a Github personal access token: https://docs.gitlab.com/user/project/import/github/
- Now create a new project as import project. Select the cloned repo, be sure to change from root to your group before to proceed
