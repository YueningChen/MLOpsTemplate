# Part 4: Continuous Integration (CI)

## Pre-requisites
- Complete [Part 0](part_0.md), [Part 1](part_1.md), [Part 2](part_2.md), [Part 3](part_3.md)

## Summary
After learning about how GitHub can be leveraged for MLOps, your team decides to start by automating the model training and evaluation process with a CI pipeline. Continuous Integration (CI) is the process of developing, testing, integrating, and evaluating new features in a staging environment where they are ready for deployment and release. 

## Steps:

1. CI pipeline template is defined in ```.github/workflows/yc_ci_train.yml```. We need to configure the secrets and environment variables (the training pipeline runs in staging environment) 

    > Action Items: Update the `yc_ci_train.yml` file with your secret credentials. 
    > - Locate the file named `yc_training_unit_test.yml` in the `.github/workflows` folder
    > - Make the following updates to the file: 
    >     - Update the secret name by replacing the ```MY_AZURE_CREDENTIALS``` to match the GitHub secret name for your Service Principal that was created in Part 0. (It most likely has a name similar to ```AZURE_CREDENTIALS_USERNAME```.)
    
    Workflows in yc_ci_train.yml loads environment viariables from .env files.

    > Action Items: Configure the variables for development enviroment in `src/workshop/env/.env.staging`
    > - `group`: resource group of the AML staging workspace.
    > - `workspace`: the AML staging workspace.
    > - `location`: the location of the AML staging workspace.
    > - `compute`: the name of the compute cluster in staging.
    > - `endpoint`: the name of the model endpoint in staging. This name needs to be unique within the region you are deploying into as the endpoint name is part of the endpoint URI.
    > - `model`: the model name.


2. Approve and merge the Pull Request created in Part_4 to integration branch. This will trigger the training in yc-ci-train workflow.

    > Action Items: Merge the Pull Request created by yc-training-unit-test workflow.
    > - Go to your browser and go to your repository. 
    > - Click on "Pull requests" tab and click on the new PR "An automatically created PR by successful unit test to integration". 
    > - Reviw the commits into `integration` branch from `yourname-dev`.
    > - Click on "Merge pull request".
    > - Click on "Confirm merge".
    
    As a reminder, integration branch is a branch which is as up to date as the main branch but we use it to train the model. Here we made some changes to the model, and we want to train and evaluate the new model. If the evaluation fails, it won't trigger the CD process and making changes to the main branch where our production code lives.

3. The merge to the integration branch triggers the yc-ci-train workflow. Click on the Actions tab on your repository and you will see CI workflow running after a few minutes. Click and examine all the steps, note that the CI Workflow is running following the steps in the ```yc-ci-train.yml``` file which you located earlier. Note that in the first few lines of this file we have defined the workflow to be triggered when a pull request is merged in the "integration" branch.

    The CI workflow has multiple steps, including setting up python version, installing libraries needed, logging in to Azure and running the training model pipeline and evaluating the model. As a part of this workflow, the updated model from our current changes is compared to our best previous model and if it performs better it passes the evaluation step (more details below).

    You can check out different steps of the training pipeline under: ```/src/workshop/pipelines/training_pipeline.yml```. 
    
    >Note: At this point, it takes about 10 minutes for the pipeline to run.
    
    If all steps pass (you can check the status under the actions in the repository), a new model is registered to AML model registry, and the new model will be deployed to staging environment (described in Part_5). If the workflow fails, there could be a few different reasons, you can open the workflow steps on the actions tab of the repository and examine it. Most likely if it fails in this case is due to the evaluation part, where our new model performs worse than our best previous model and doesn't pass the evaluation step and the whole workflow fails. To resolve that continue reading the following section.

> OPTIONAL READING: For the evaluation and comparison of the current model with our best previous model, we have included some code in the following script: ```/src/workshop/core/evaluating/ml_evaluating.py```. Note that on line 85 of the script we are comparing the R-square of the current model with our best previous model in order to decide if we want to allow any changes to the model and main branch. You might want to edit this and relax it a little bit in order for the evaluation step to pass if you already have a really good model registered. Note that you can change the evaluation metrics based on your actual use case in the future.


## Success criteria
- Trigger CI workflow when a pull request is merged to the integration branch
- Successfully run the CI workflow which also includes the AML pipeline

## Reference materials

- [GitHub Actions](https://github.com/features/actions)
- [GitHub Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-token)
- [GitHub Actions Workflow Triggers](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [Azure ML CLI v2](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-train-cli)
- [Azure ML CLI v2 Examples](https://github.com/Azure/azureml-examples/tree/main/cli)


## [Go to Part 5](part_5.md)

