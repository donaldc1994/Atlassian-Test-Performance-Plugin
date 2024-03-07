# Jira Data Center Apps Performance Toolkit: User Guide
### Enterprise-Scale Environment
- This document provides comprehensive guidance on setting up an enterprise-scale environment for Jira Data Center using the Atlassian Marketplace Data Center Apps Performance Toolkit.

### ! IMPORTANT  : 

1. <strong>Create Virtual environment</strong>

    Creating a virtual environment in Python allows you to isolate dependencies for different projects, which helps prevent conflicts between packages and keeps your system clean. Here's a step-by-step guide on how to create a virtual environment in Python:

    - <strong>Step 1 : Create a Virtual Environment</strong>
        
            python3.9 -m venv myenv
        
    - <strong>Step 2 : Activate the Virtual Environment</strong>
    
     ```
            On Windows:

            myenv\Scripts\activate
    ```
            On macOS and Linux:
            
            source myenv/bin/activate
        
    Once activated, you should see the name of the virtual environment in your command prompt or terminal to indicate that you are now working within the virtual environment. 
    - <strong>Step 3 : You can install now the required libraries on the requirements.txt file inside the project. </strong>

            pip3 install --upgrade pip
            pip install -r requirements.txt



### Termination of Development Environment
- Before initiating the creation of an enterprise-scale environment, ensure the complete termination of the development environment. Follow the provided instructions for termination. 
- Should any issues arise during uninstallation, utilize the "Force terminate" command.

### Setting Up Jira Data Center Enterprise-Scale Environment
#### EC2 CPU Limitation
- For the successful installation of a 4-pods Data Center (DC) environment and execution pod, it is imperative to have a minimum of 40 vCPU cores available. 
- AWS accounts typically have CPU limitations, often set to low numbers such as 5 vCPUs per region.
- It's advisable to increase the CPU limit by using the "Request increase at account-level" option.

#### AWS Cost Estimation
- Utilize the AWS Pricing Calculator to estimate the usage charges for AWS services. 
- Prices provided are approximate and may vary based on factors such as region, instance type, and database (DB) deployment type.

[For more information, refer to the AWS Cost Estimation guide](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#aws-cost-estimation).

# Setup of Jira Data Center on Kubernetes (K8s)
- This section details the setup process for a Jira Data Center enterprise-scale environment on Kubernetes (K8s). It outlines the data dimensions and values required for an enterprise-scale dataset. All datasets are configured to use standard admin/admin credentials.

- For detailed instructions on setting up Jira Data Center on Kubernetes, please refer [here](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#setup-jira-data-center-enterprise-scale-environment-on-k8s).

## Installation of Enterprise-Scale Jira Data Center with Large Dataset
The installation process involves several steps, including the creation of access keys for the IAM user, cloning of the Data Center App Performance Toolkit locally, and setting AWS access keys. Additionally, it entails the generation of a new trial license on Atlassian, overriding the deployment version, and starting the installation from the local terminal.

#### [Reference Link](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#enterprise-scale-environment)

### STEP 1 : Install enterprise-scale Jira DC with "large" dataset included
To install enterprise-scale Jira DC with a large dataset:
-  Create access keys for an IAM user and create an admin user instead of root user credentials.
-  Clone the Data Center App Performance Toolkit locally and set AWS access keys in the aws_envs file. 
- Set required variables in the dcapt.tfvars file, including environment_name, products, jira_license, region, and trial license.

Start the installation from a local terminal (Git Bash for Windows users) using the command 

        docker run --pull=always --env-file aws_envs \
        -v "/$PWD/dcapt.tfvars:/data-center-terraform/conf.tfvars" \
        -v "/$PWD/dcapt-snapshots.json:/data-center-terraform/dcapt-snapshots.json" \
        -v "/$PWD/logs:/data-center-terraform/logs" \
        -it atlassianlabs/terraform:2.7.1./install.sh -c conf.tfvars"

Copy the product URL from the console output and ensure all datasets use standard admin/admin credentials. Change the default password from the UI account page for security reasons.

Set up load configuration for enterprise-scale runs by checking the jira.yml configuration file and making sure parameters are changed back to the defaults. The load_executor should be jmeter or locust, with concurrency set to 200, test duration of 45m, ramp-up of 3m, and total actions per hour of 54500. Run the toolkit for each test scenario in the next section.

### STEP 2 : Setting up load configuration for Enterprise-scale runs
* TerraForm deployment configuration includes a dedicated execution environment pod for tests.
* Check jira.yml configuration file for changes made for dev runs.
* Copies application_hostname, application_protocol, application_port, secure, application_postfix, admin_login, admin_password, load_executor, concurrency, test_duration, ramp-up, and total_actions_per_hour are required.
* Concurrent virtual users for jmeter or locust scenario are 200.
* Test duration is 45m, ramp-up is 3m, and total actions per hour is 54500.
* Toolkit for each test scenario is required.
    ```
    application_hostname: test_jira_instance.atlassian.com   # Jira DC hostname without protocol and port e.g. test-jira.atlassian.com or localhost
    application_protocol: http      # http or https
    application_port: 80            # 80, 443, 8080, 2990, etc
    secure: True                    # Set False to allow insecure connections, e.g. when using self-signed SSL certificate
    application_postfix: /jira      # e.g. /jira for TerraForm deployment url like `http://a1234-54321.us-east-2.elb.amazonaws.com/jira`. Leave this value blank for url without postfix.
    admin_login: admin
    admin_password: admin
    load_executor: jmeter           # jmeter and locust are supported. jmeter by default.
    concurrency: 200                # number of concurrent virtual users for jmeter or locust scenario
    test_duration: 45m
    ramp-up: 3m                     # time to spin all concurrent users
    total_actions_per_hour: 54500   # number of total JMeter/Locust actions per hour
    ```

### STEP 3 : Running the test scenarios from execution environment pod against enterprise-scale Jira Data Center
The Data Center App Performance Toolkit is utilized for performance and scale testing of your app, involving two test scenarios: performance regression and scalability testing, each requiring multiple test runs.

### [Scenario 1: Performance Regression](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#scenario-1--performance-regression)

Scenario 1 focuses on performance regression to identify basic performance issues without the need to spin up a multi-node Jira DC. To perform this, follow these steps:

1. <strong>[Run 1](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#run-1---50-min-)</strong> (~50 min) 

    To receive performance baseline results <strong>without an app installed</strong>. Make sure jira.yml and toolkit code base have default configuration from the master branch. Check load configuration parameters needed for enterprise-scale runs. Check the correctness of application_hostname, application_protocol, application_port, and application_postfix in the.yml file.

        export ENVIRONMENT_NAME=your_environment_name

        docker run --pull=always --env-file ./app/util/k8s/aws_envs \
        -e REGION=us-east-2 \
        -e ENVIRONMENT_NAME=$ENVIRONMENT_NAME \
        -v "/$PWD:/data-center-terraform/dc-app-performance-toolkit" \
        -v "/$PWD/app/util/k8s/bzt_on_pod.sh:/data-center-terraform/bzt_on_pod.sh" \
        -it atlassianlabs/terraform:2.7.1 bash bzt_on_pod.sh jira.yml

2. <strong>[Run 2](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#run-2---50-min---lucene-index-timing-test-)</strong> (~50 min + Lucene Index timing test)

    To conduct a foreground re-index on a single-node Data Center deployment with your app installed and a dataset with 1M issues. The re-index time for Jira is about 50-70 minutes. Benchmark your re-index time <strong>with your app installed</strong> by installing the app you want to test, setting up app license, and going to cog icon > System > Indexing.

        export ENVIRONMENT_NAME=your_environment_name

        docker run --pull=always --env-file ./app/util/k8s/aws_envs \
        -e REGION=us-east-2 \
        -e ENVIRONMENT_NAME=$ENVIRONMENT_NAME \
        -v "/$PWD:/data-center-terraform/dc-app-performance-toolkit" \
        -v "/$PWD/app/util/k8s/bzt_on_pod.sh:/data-center-terraform/bzt_on_pod.sh" \
        -it atlassianlabs/terraform:2.7.1 bash bzt_on_pod.sh jira.yml

3. Take a screenshot of the acknowledgment screen displaying the re-index time and Lucene index timing. If the window is not displayed, log in to Jira one more time and navigate to cog icon > System > Indexing.

4. <strong>[Generate a performance regression report](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#generating-a-performance-regression-report)</strong> 

    By editing the./app/reports_generation/performance_profile.yml file and running the command from local terminal (Git Bash for Windows users). In the./app/results/reports/YY-MM-DD-hh-mm-ss folder, view the.csv file,.png chart file, and performance scenario summary report. If there is an impact (>20%) on any action timing, it is recommended to look into the app implementation to understand the root cause of this delta.

        docker run --pull=always \
        -v "/$PWD:/dc-app-performance-toolkit" \
        --workdir="//dc-app-performance-toolkit/app/reports_generation" \
        --entrypoint="python" \
        -it atlassian/dcapt csv_chart_generator.py performance_profile.yml

In conclusion, these scenarios help identify basic performance issues without the need to spin up a multi-node Jira DC. By following these steps, you can ensure that your app does not have any performance impact when not exercised.

### [Scenario 2: Scalability testing](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#scenario-2--scalability-testing)

-  The purpose of scalability testing is to reflect the impact on the customer experience when operating across multiple nodes. To demonstrate performance impacts of operating your app at scale, it is recommended to test your Jira DC app in a cluster.

1. <strong>[Run 3](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#run-3---50-min-)</strong> (~50 min) 

    To receive scalability benchmark results for one-node Jira DC <strong>with app-specific actions</strong>, ensure that the jira.yml and toolkit code base has code base with your developed app-specific actions. Check the correctness of application_hostname, application_protocol, application_port, and application_postfix in the.yml file and check load configuration parameters needed for enterprise-scale runs.

          export ENVIRONMENT_NAME=your_environment_name

          docker run --pull=always --env-file ./app/util/k8s/aws_envs \
          -e REGION=us-east-2 \
          -e ENVIRONMENT_NAME=$ENVIRONMENT_NAME \
          -v "/$PWD:/data-center-terraform/dc-app-performance-toolkit" \
          -v "/$PWD/app/util/k8s/bzt_on_pod.sh:/data-center-terraform/bzt_on_pod.sh" \
          -it atlassianlabs/terraform:2.7.1 bash bzt_on_pod.sh jira.yml


2. <strong>[Run 4](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#run-4---50-min-)</strong> (~50 min)

    To ensure that AWS vCPU limit is not lower than the needed number. For <strong>two-node Jira DC with app-specific actions</strong>, navigate to the dc-app-performance-toolkit folder and start tests execution. Review results_summary.log file under artifacts dir location and ensure that overall status is OK before moving to the next steps. For an enterprise-scale environment run, the acceptable success rate for actions is 95% and above.
 ```
        1- Navigate to dc-app-performance-toolkit/app/util/k8s folder.
        2- Open dcapt.tfvars file and set jira_replica_count value to 2.
        3- From local terminal (Git Bash for Windows users) start scaling (~20 min)

        docker run --pull=always --env-file aws_envs \
        -v "/$PWD/dcapt.tfvars:/data-center-terraform/conf.tfvars" \
        -v "/$PWD/dcapt-snapshots.json:/data-center-terraform/dcapt-snapshots.json" \
        -v "/$PWD/logs:/data-center-terraform/logs" \
        -it atlassianlabs/terraform:2.7.1 ./install.sh -c conf.tfvars
 ```
  

        - Navigate to dc-app-performance-toolkit folder and start tests execution:

          export ENVIRONMENT_NAME=your_environment_name

          docker run --pull=always --env-file ./app/util/k8s/aws_envs \
          -e REGION=us-east-2 \
          -e ENVIRONMENT_NAME=$ENVIRONMENT_NAME \
          -v "/$PWD:/data-center-terraform/dc-app-performance-toolkit" \
          -v "/$PWD/app/util/k8s/bzt_on_pod.sh:/data-center-terraform/bzt_on_pod.sh" \
          -it atlassianlabs/terraform:2.7.1 bash bzt_on_pod.sh jira.yml


3. <strong>[Run 5](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#run-5---50-min-)</strong> (~50 min) 

    To ensure that AWS vCPU limit is not lower than the needed number. For <strong>four-node Jira DC with app-specific actions</strong>, scale your Jira Data Center deployment to four nodes as described in Run 4.

           export ENVIRONMENT_NAME=your_environment_name
           docker run --pull=always --env-file ./app/util/k8s/aws_envs \
           -e REGION=us-east-2 \
           -e ENVIRONMENT_NAME=$ENVIRONMENT_NAME \
           -v "/$PWD:/data-center-terraform/dc-app-performance-toolkit" \
           -v "/$PWD/app/util/k8s/bzt_on_pod.sh:/data-center-terraform/bzt_on_pod.sh" \
           -it atlassianlabs/terraform:2.7.1 bash bzt_on_pod.sh jira.yml

4. <strong>[Generate a scalability report](https://developer.atlassian.com/platform/marketplace/dc-apps-performance-toolkit-user-guide-jira/#generating-a-report-for-scalability-scenario)</strong>

    By editing the./app/reports_generation/scale_profile.yml file and inserting the relative path to the results directory of Run 3. In the./app/results/reports/YY-MM-DD-hh-mm-ss folder, view the.csv file,.png chart file, and performance scenario summary report. If there is an impact (>20%) on any action timing, take a look into the app implementation to understand the root cause of this delta.

           docker run --pull=always \
           -v "/$PWD:/dc-app-performance-toolkit" \
           --workdir="//dc-app-performance-toolkit/app/reports_generation" \
           --entrypoint="python" \
           -it atlassian/dcapt csv_chart_generator.py scale_profile.yml

It is recommended to terminate an enterprise-scale environment after completing all tests and follow the Terminate enterprise-scale environment instructions. If any problems with uninstall use the Force terminate command.

Attach performance testing results to your ECOHELP ticket by creating two reports folders: one with performance profile and the second with scale profile results. Each folder should have profile.csv, profile.png, profile_summary.log, and profile.csv_chart_generator.py files.
