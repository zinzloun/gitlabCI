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
- Now create a new project as import project. Select the cloned repo, be sure to change the owner from root to your group before to proceed
- We need to create another branch named master, since SonarCloud free plan allows only to scan master branches. In your new project navigate to Code->Branches, click New branch, set the name as master and create it from main. Then go to Settings->Repository, expand Branch defaults and set master. Eventually you can return to the branches section to delete main

## Create project variable
You need to create a SonarCloud account, then you must create an organization and  related project. 
Visit https://sonarcloud.io/projects to create both. We will need of these information (organization and project key) to configure the scan, plus of course the token. 
To get a token for the pipe visit https://sonarcloud.io/account/security

Once you get the token create a variable for the project in Gitlab. Go to Setting -> CI/CD, then expand Variables and add a new one, having the following set:
- Visibility: masked and hidden
- Uncheck Protect variable
- Key: SONAR_TOKEN
- Value: <your-token>

Do the same for Snyk, in this case you need just of your Auth token, that you can get, once you have created an account, at https://app.snyk.io/account.
In Gitlab create another variable as done before, set the Key value as SNYK_TOKEN and the value as you <Auth-token>

## Create the CI pipe
Our pipe will be composed by thw following stages:
- sast-snyk: performs SAST analysis using Snyk
- snyk-deps: performs SCA analysis using SnyK
- sast-sonar: performs SAST analysis using SonarCloud
- build: builds and run extremely-vulnerable-flask-app using a Docker container
- dast: Perform DAST analysis using OWASP Zap against extremely-vulnerable-flask-app 
- cleanup: clean up docker artifacts
To run the pipe copy from this repository to root of the Gitlab project the <b>sonar.properties</b> file, change the properties values according to your SonarCloud project configuration, then copy the <b>.gitlab-ci.yml</b> configuration. The pipe will start.

## Security considerations
- If your pipe is compromised the attack payloads will run on the gitlab-runner user context, so grant to this user only the minimum privileges to operate, don't add the user to the sudoers
- In a real enviroment the pipe should run in a developer branch, then create another pipe for the merge request to be approved by another group, ideally the developers should not have the rights to merge requests
- Secure Docker engine, more information can be found here: https://docs.docker.com/engine/security
