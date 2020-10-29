# twobranchworkflow - External repository for Divio Continuous deployment with Github Actions

Setting up a deployment service environment for continuous deployment using Github Actions.
 
* Create a new repository on Github with a master branch.

* Create a new project on [Divio control panel](https://control.divio.com/control/) with the new github repository as your external git server 

* Add the SSH public key provided by Divio to your Github repository **Settings**, **Deploy Key** and check the **Allow write access** to allow Divio to push in to the repository.

* Configure Webhooks for the github to send signals when commits are made. See how to configure webhooks in [Divio Developer Handbook](https://docs.divio.com/en/latest/how-to/resources-configure-git/#configure-a-webhook-for-the-git-repository).

* Setup the Secret Keys to be able to login to Divio. In your github repository, go to **Secrets** from the **Settings** tab and add **New secrets** of your Divio access token and your **project-id** (from Divio control panel url `https://control.divio.com/control/<organisation-id>/edit/<project-id>/` as `DIVIO_API_TOKEN` and `DIVIO_PROJECT_ID` environment variables respectively.

Setup your workflow
-------------------
Open the **Actions** tab, you may choose from the existing continuous deployment workflows, but for now **skip to set up a workflow yourself** and a `main.yml` file will be created for you. You can rename it anything you want. 

Commit your yaml file with the following configuration.

    # This workflow will install Divio CLI and deploy into Divio on a push event
    
    name: Divio Continuous Deployment
    
    # push event to master branch triggers the workflow
    on:
    push:
      branches: [ master ]
    
    # jobs run sequentially or in parallel to execute the workflow
    jobs:
    
    # This workflow contains a single job called "deploy"
    deploy:
      # Setting environment variables of the secret keys for easy access
      env:
        DIVIO_API_TOKEN: ${{ secrets.DIVIO_API_TOKEN }}
        DIVIO_PROJECT_ID: ${{ secrets.DIVIO_PROJECT_ID }}  
        # DIVIO_PROJECT_SLUG: ${{ secrets.DIVIO_PROJECT_SLUG }}
      # The type of runner that this job will run on
      runs-on: ubuntu-latest
      # Steps represent a sequence of tasks that will be executed as part of the job
      steps:
      # Checks-out the repository under $GITHUB_WORKSPACE, so this job can access it
      - uses: actions/checkout@v2
      # setting up Python 3.8 to be able to run pip to install Divio CLI
      - name: Set up Python 3.8
        uses: actions/setup-python@v1
        with:
          python-version: 3.8
    
      # - name: Deploy to Divio Cloud using the API
      #   run: |
      #     curl -X POST --data 'stage="$DIVIO_PROJECT_SLUG"' --header "Authorization: Basic "$DIVIO_API_TOKEN"" https://control.divio.com/api/v1/website/"$DIVIO_PROJECT_ID"/deploy/
  
      - name: Install Divio CLI
        run: |
          python -m pip install --upgrade pip
          pip install divio-cli
  
      # Login to Divio using the access token and deploy
      - name: Deploy to Divio
        run: |
          divio login "$DIVIO_API_TOKEN"
          divio project deploy --remote-id "$DIVIO_PROJECT_ID"


On the terminal: 
* Create and git initialize a directory 
* Clone your github repository
* Develop your applications and 
* `git push` to deploy to Divio.
