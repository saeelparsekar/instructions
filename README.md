# pw-poc

## Prerequisites

1. [Install Salesforce CLI](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_install_cli.htm)

2. [Set Up LWC Local Development](https://developer.salesforce.com/tools/vscode/en/localdev/set-up-lwc-local-dev)

```
sfdx plugins:install @salesforce/lwc-dev-server
```

## Building and publishing

The following steps will build and publish the packages:

- clone or download the repo;
- run `yarn` to install dependencies;

To start the local server use `yarn start`.

# Setup

# Prerequisites for Setup

## Create a Developer Edition Salesforce Org
- Create a new Developer Edition Org by visiting the following link:
https://developer.salesforce.com/signup

## Enable Dev Hub and Unlocked Packages
- On homescreen, click on gear icon on the top-right, Click on Setup, inside the quick search type in **Dev Hub** and click on it - Enable Dev Hub, Enable Source Tracking in Developer and Developer Pro Sandboxes, and Enable Unlocked Packages and Second-Generation Managed Packages for your org

## Enable Digital Experiences
- From Setup, enter Digital Experiences in the Quick Find box, then select Digital Experiences | Settings.
- Select Enable Digital Experiences.
- Select a domain name, and click Check Availability to make sure that it’s not already in use. We suggest that you use something recognizable to your users, such as your company name. The domain name is the same for all experiences. You create a unique URL for each one when creating it by entering a unique name at the end of the URL. For example, if your domain name is UniversalTelco.my.site.com and you’re creating a customer community, you can enter customers to create the unique URL UniversalTelco.my.site.com/customers.
- Click Save.

## Enable ExperienceBundle Metadata API
- From Setup, enter Digital Experiences in the Quick Find box, then select Digital Experiences | Settings
- Enable **Enable ExperienceBundle Metadata API** under Experience Management Settings. Click Save. (Click OK on the alert)

## Use System Admin Email 
- In the following files replace "update with sysadmin email of org" with System Admin Email of Org where the project will be deployed
1. force-app\main\default\approvalProcesses\CMS_Page__c.CMS_Page_Approval.approvalProcess-meta.xml
2. force-app\main\default\queues\Approver_CMS_Page.queue-meta.xml
3. force-app\main\default\queues\Publisher_CMS_Page.queue-meta.xml
4. force-app\main\default\sites\PartnerWorld.site-meta.xml
5. force-app\main\default\sites\MPW.site-meta.xml
6. force-app\main\default\queues\Business_Controls_Reviewers.queue-meta.xml
7. force-app\main\default\queues\External_Regulatory_Officers.queue-meta.xml
8. force-app\main\default\queues\GPM_Reviewers.queue-meta.xml
9. force-app\main\default\queues\Legal_Reviewers.queue-meta.xml
10. force-app\main\default\queues\Trust_And_Compliance_Reviewers.queue-meta.xml
11. force-app\main\default\workflows\CMS_Page__c.workflow-meta.xml
12. force-app\main\default\approvalProcesses\CMS_Page__c.Enhanced_CMS_Page_Approval.approvalProcess-meta.xml
13. force-app\main\default\connectedApps\MPW_Page_Migration.connectedApp-meta.xml

# Deployment

---------------------------------------------------------------------------------------
# APPROACH 1 - UNLOCKED PACKAGES

### Note - The package version which will be included in the PR must be created from the base Org ONLY. 
### The installation key for version installation is mentioned in the vault. Refer to the given link:- https://ibm.ent.1password.com/vaults/g7a2gocfqv5kvt4smnfhuzso2i/allitems/7vcezjmufhlm5xctau4x7x3ks4
# Setup for New or first time users

## Install and Test the Package Version in a Dev org
1. After creating a Developer Edition account. Authorize your dev org inside VS Code using `sfdx auth:web:login -d -a DevHubName` 
   where `DevHubName` is the alias you wish to set for your dev org
2. Install the latest package into Dev org using
   `sfdx force:package:install --wait 10 --publishwait 10 --package PWPOC@0.0.1-1 --installationkey test1234 --noprompt -u DevHubName`
   Where `PWPOC@0.0.1-1` is the latest package alias from `sfdx-project.json` file and `test1234` is the installation key. Please replace it with appropriate installation key.
3. Assign the permission set.
    `sfdx force:user:permset:assign -n CMS_MPW_Authoring`
4. Open your dev org using `sfdx force:org:open -u DevHubName` and login with your credentials.
5. Disable the `UpdateSectionOrder` trigger. In the Dev Org
   - Navigate to setup
   - Type Apex Triggers into the Quick Find
   - Click the link for Apex Triggers
   - click edit next to the trigger you want to siaable
   - Uncheck the Is Active checkbox
   - Hit save
6. Import test data into your org by running `sfdx force:data:tree:import --plan ./data/plan.json`
7. Import Personalization (CMS Pages, Audiences, Audience rules) data into your org by running `sfdx force:data:tree:import --plan ./data-audiences/plan.json`
8. Import test incentives data (including assets) into your org by running `node ./data-incentive/load-data.js` from the root directory (Ensure the default org is set to the Dev org)
9. After successfully importing the data in the above 2 json files, Enable the `UpdateSectionOrder` trigger. In the Dev Org
   - Navigate to setup
   - Type Apex Triggers into the Quick Find
   - Click the link for Apex Triggers
   - click edit next to the trigger you want to enable
   - Check the Is Active checkbox
   - Hit save
10. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\unlockPackageXmlDeployment.forceignore` file.
11. The unlocked package does not include all the files, hence the files which were not a part of the unlocked package are present in manifest\unlockPackage.xml and need to be deployed to the org by either running `sfdx force:source:deploy -x manifest/unlockPackage.xml` or by right clicking on manifest\unlockPackage.xml and selecting the option "Deploy Source in Manifest to Org".
12. To publish PartnerWorld experience site run `sfdx force:community:publish --name PartnerWorld`
13. To publish MPW experience site run `sfdx force:community:publish --name MPW`


## Install and Test the Package Version in a Scratch Org
1. After creating a Developer Edition account. Authorize your dev org inside VS Code using `sfdx auth:web:login -d -a DevHubName` where `DevHubName` is the alias you wish to set for your dev org
2. Open your dev org using `sfdx force:org:open -u DevHubName` and login with your credentials
3. Create the scratch org with setDefaultusername.
      `sfdx force:org:create --setdefaultusername --definitionfile config/project-scratch-def.json -a ScratchOrgName -d 7`
   where `ScratchOrgName` is the name of the new scratch org you want to open / create.
4. Set a Default Org to your scratch org to install package in correct org
   `sfdx force:config:set defaultusername=ScratchOrgName`
5. Install the package into the scratch org using the alias for the package version.
   `sfdx force:package:install --wait 10 --publishwait 10 --package PWPOC@0.0.1-1 --installationkey test1234 --noprompt -u ScratchOrgName`
   Where `PWPOC@0.0.1-1` is the latest package alias from `sfdx-project.json` file
   and `test1234` is the installation key. Please replace it with appropriate installation key.
6. Assign the permission set.
    `sfdx force:user:permset:assign -n CMS_MPW_Authoring -u ScratchOrgName`
7. Open the scratch org by running `sfdx force:org:open -u ScratchOrgName`
8. Disable the `UpdateSectionOrder` trigger. In the ScratchOrg
   - Navigate to setup
   - Type Apex Triggers into the Quick Find
   - Click the link for Apex Triggers
   - click edit next to the trigger you want to siaable
   - Uncheck the Is Active checkbox
   - Hit save
9. Import test data into your org by running `sfdx force:data:tree:import --plan ./data/plan.json -u ScratchOrgName`
10. Import Personalization (CMS Pages, Audiences, Audience rules) data into your org by running `sfdx force:data:tree:import --plan ./data-audiences/plan.json -u ScratchOrgName`
11. Import test incentives data (including assets) into your org by running `node ./data-incentive/load-data.js` from the root directory (Ensure the default org is set to the scratch org)
12. After successfully importing the data in the above 2 json files, Enable the `UpdateSectionOrder` trigger.  In the ScratchOrg
   - Navigate to setup
   - Type Apex Triggers into the Quick Find
   - Click the link for Apex Triggers
   - click edit next to the trigger you want to enable
   - Check the Is Active checkbox
   - Hit save
13. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\unlockPackageXmlDeployment.forceignore` file.
14. Push source to your org by running `sfdx force:source:push -u ScratchOrgName`
15. To publish PartnerWorld experience site run `sfdx force:community:publish --name PartnerWorld -u ScratchOrgName`
16. To publish MPW experience site run `sfdx force:community:publish --name MPW -u ScratchOrgName`
17. The list of scratch orgs can be viewed by running `sfdx force:org:list` - the newly created scratch org will be listed here as well.

---------------------------------------------------------------------------------------

# Setup for existing users/developers

Make sure you have the latest code from main merged into your feature branch.
Here is it considered that the Dev Org and Scratch org is created and project is already setup into your respective Dev Org or Scratch Org.

## 1. Install and Test the Package Version in a Dev org

1. Install the latest package into Dev org using
   `sfdx force:package:install --wait 10 --publishwait 10 --package PWPOC@0.0.1-1 --installationkey test1234 --noprompt -u DevHubName`
   Where `PWPOC@0.0.1-1` is the latest package alias from `sfdx-project.json` file
   and `test1234` is the installation key. Please replace it with appropriate installation key.
2. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\unlockPackageXmlDeployment.forceignore` file.
11. The unlocked package does not include all the files, hence the files which were not a part of the unlocked package are present in manifest\unlockPackage.xml and need to be deployed to the org by either running `sfdx force:source:deploy -x manifest/unlockPackage.xml` or by right clicking on manifest\unlockPackage.xml and selecting the option "Deploy Source in Manifest to Org".

## 2. Create the Unlocked Package
Now make the changes in your feature branch for the feature which is being developed.
1. Open `sfdx-project.json`
2. Update the sourceApiVersion to match the version of the Salesforce CLI
3. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\unlockPackageCreation.forceignore` file.
4. Create Package
  `sfdx force:package:create -t Unlocked -r force-app -n PWPOC --orgdependent --targetdevhubusername DevHubName`
   Please replace 'PWPOC' with your PackageName. 
   This command will update `sfdx-project.json` file

## 3. Create the Package Version
Whenever a version of the existing package has to be created to include the changes after the package was created, we can create a package version by following the steps gien below.
1. Open the `sfdx-project.json` file
2. Update the versionName and versionNumber according to the convention which is being followed during the sprint. Currently "versionName": "ver 0.0.1" and "versionNumber": "0.0.1.NEXT" is being followed.
3. Save `sfdx-project.json`
4. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\unlockPackageCreation.forceignore` file.
5. Create your first package version
   `sfdx force:package:version:create --package PWPOC --installationkey test1234 --wait 10 --targetdevhubusername DevHubName`
   Where `DevHubName` is the alias of your dev org. Please replace 'PWPOC' with your PackageName and `test1234` is the installation key. Please replace it with appropriate installation key.
   This command will update `sfdx-project.json` file

## 4. Install and Test the Package Version Changes for the feature branch in a New Scratch org

1. After creating a Developer Edition account. Authorize your dev org inside VS Code using `sfdx auth:web:login -d -a DevHubName` where `DevHubName` is the alias you wish to set for your dev org
2. Open your dev org using `sfdx force:org:open -u DevHubName` and login with your credentials
3. Create the scratch org with setDefaultusername.
      `sfdx force:org:create --setdefaultusername --definitionfile config/project-scratch-def.json -a ScratchOrgName -d 7`
   where `ScratchOrgName` is the name of the new scratch org you want to open / create.
4. Set a Default Org to your scratch org to install package in correct org
   `sfdx force:config:set defaultusername=ScratchOrgName`
5. Install the package into the scratch org using the alias for the package version.
   `sfdx force:package:install --wait 10 --publishwait 10 --package PWPOC@0.0.1-1 --installationkey test1234 --noprompt -u ScratchOrgName`
   Where `PWPOC@0.0.1-1` is the latest package alias from `sfdx-project.json` file
   and `test1234` is the installation key. Please replace it with appropriate installation key.
6. Assign the permission set.
    `sfdx force:user:permset:assign -n CMS_MPW_Authoring -u ScratchOrgName`
7. Open the scratch org by running `sfdx force:org:open -u ScratchOrgName`
8. Disable the `UpdateSectionOrder` trigger. In the ScratchOrg
   - Navigate to setup
   - Type Apex Triggers into the Quick Find
   - Click the link for Apex Triggers
   - click edit next to the trigger you want to siaable
   - Uncheck the Is Active checkbox
   - Hit save
9. Import test data into your org by running `sfdx force:data:tree:import --plan ./data/plan.json -u ScratchOrgName`
10. Import Personalization (CMS Pages, Audiences, Audience rules) data into your org by running `sfdx force:data:tree:import --plan ./data-audiences/plan.json -u ScratchOrgName`
11. Import test incentives data (including assets) into your org by running `node ./data-incentive/load-data.js` from the root directory (Ensure the default org is set to the scratch org)
12. After successfully importing the data in the above 2 json files, Enable the `UpdateSectionOrder` trigger.  In the ScratchOrg
   - Navigate to setup
   - Type Apex Triggers into the Quick Find
   - Click the link for Apex Triggers
   - click edit next to the trigger you want to enable
   - Check the Is Active checkbox
   - Hit save
13. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\unlockPackageXmlDeployment.forceignore` file.
14. Push source to your org by running `sfdx force:source:push -u ScratchOrgName`
15. To publish PartnerWorld experience site run `sfdx force:community:publish --name PartnerWorld -u ScratchOrgName`
16. To publish MPW experience site run `sfdx force:community:publish --name MPW -u ScratchOrgName`
17. The list of scratch orgs can be viewed by running `sfdx force:org:list` - the newly created scratch org will be listed here as well.

---------------------------------------------------------------------------------------
# APPROACH 2 - DEPLOY ON DEV ORG
# Note before deploying
1. In the following file replace `Active` with `Inactive`
    - force-app\main\default\triggers\UpdateSectionOrder.trigger-meta.xml
2. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\packageXmlDeployment.forceignore` file.

1. The file manifest\package.xml needs to be deployed to the org by either running `sfdx force:source:deploy -x manifest/package.xml` or by right clicking on manifest\package.xml and selecting the option "Deploy Source in Manifest to Org"
2.  Apply the `CMS_MPW_Authoring` permission set for the current user by running `sfdx force:user:permset:assign -n CMS_MPW_Authoring`
3. Import data 
   - Run the following command in the terminal to import the demo pages
 data
     `sfdx force:data:tree:import --plan ./data/plan.json`
   - Run the following command in the terminal to import the CMS Pages, Audiences, Audience rules related to Personalization
     `sfdx force:data:tree:import --plan ./data-audiences/plan.json`
   - Run the following command in the terminal from the root directory to import the incentives data (including assets)
     `node ./data-incentive/load-data.js` (Ensure the default org is set to the dev org)
4. After successfully importing the data in the above 2 json files, activate the following trigger
  - In the following file replace `Inactive` with `Active`, save and deploy the file
    - force-app\main\default\triggers\UpdateSectionOrder.trigger-meta.xml
5. Deploy experience site
  - Search for 'Experience site related files' in `.forceignore` file and comment the  files under it. Namely, 
      - force-app/main/default/experiences
      - force-app/main/default/navigationMenus
      - force-app/main/default/networks
      - force-app/main/default/networkBranding
      - force-app/main/default/sites
  - The file manifest\package.xml needs to be deployed to the org by either running `sfdx force:source:deploy -x manifest/package.xml` or by right clicking on manifest\package.xml and selecting the option "Deploy Source in Manifest to Org"
6. Deploy the Guest Profiles
  - Search for 'Guest Profiles' in `.forceignore` file and comment the  files under it. Namely, 
      - force-app\main\default\profiles\MPW Profile.profile-meta.xml
      - force-app\main\default\profiles\PartnerWorld Profile.profile-meta.xml
      - force-app\main\default\sharingRules\CMS_Page__c.sharingRules-meta.xml
   - The file manifest\package.xml needs to be deployed to the org by either running `sfdx force:source:deploy -x manifest/package.xml` or by right clicking on manifest\package.xml and selecting the option "Deploy Source in Manifest to Org"
7. Publish sites
   - To publish PartnerWorld experience site run `sfdx force:community:publish --name PartnerWorld`
   - To publish MPW experience site run `sfdx force:community:publish --name MPW`


---------------------------------------------------------------------------------------
# APPROACH 3 - DEPLOY ON SCRATCH ORG
### Note before deploying
1. In the following file replace `Active` with `Inactive`
    - force-app\main\default\triggers\UpdateSectionOrder.trigger-meta.xml
2. Replace the current contents of `.forceignore` file with all the content in `forceignoreFiles\packageXmlDeployment.forceignore` file.

1. Create a Developer Edition account. Authorize your dev org inside VS Code using `sfdx auth:web:login -d -a DevHubName` where `DevHubName` is the alias you wish to set for your dev org
2. Open your dev org using `sfdx force:org:open -u DevHubName` and login with your credentials.
3. On homescreen, click on gear icon on the top-right, Click on Setup, inside the quick search type in **Dev Hub** and click on it - enable Dev Hub, Source Tracking and Unlocked Packages for your org.
4. Open a terminal and type in: `sfdx force:org:create -f config/project-scratch-def.json -a ScratchOrgName -d 7` where `ScratchOrgName` is the name of the new scratch org you want to create.
5. The list of scratch orgs can be viewed by running `sfdx force:org:list` - the newly created scratch org will be listed here as well.
6. Run the following command to set the scratch org as default, this will help when pushing/retrieving from source `sfdx force:config:set defaultusername=ScratchOrg username`
7. Push source to your org by running `sfdx force:source:push -u ScratchOrgName`
8. Apply the `CMS_MPW_Authoring` permission set for the current user by running `sfdx force:user:permset:assign -n CMS_MPW_Authoring -u ScratchOrgName`.
9. Import demo pages data into your org by running `sfdx force:data:tree:import --plan ./data/plan.json -u ScratchOrgName`
10. Import Personalization (CMS Pages, Audiences, Audience rules) data into your org by running `sfdx force:data:tree:import --plan ./data-audiences/plan.json -u ScratchOrgName`
11. Import test incentives data (including assets) into your org by running `node ./data-incentive/load-data.js` from the root directory (Ensure the default org is set to the scratch org)
12. After successfully importing the data in the above 2 json files, activate the following trigger
  - In the following file replace `Inactive` with `Active`, save and deploy the file
    - force-app\main\default\triggers\UpdateSectionOrder.trigger-meta.xml
13. Search for 'Experience site related files' in `.forceignore` file and comment the  files under it. Namely, 
  - force-app/main/default/experiences
  - force-app/main/default/navigationMenus
  - force-app/main/default/networks
  - force-app/main/default/networkBranding
  - force-app/main/default/sites
14. Push source to your org by running `sfdx force:source:push -u ScratchOrgName`
15. Search for 'Guest Profiles' in `.forceignore` file and comment the  files under it. Namely, 
    - force-app\main\default\profiles\MPW Profile.profile-meta.xml
    - force-app\main\default\profiles\PartnerWorld Profile.profile-meta.xml
    - force-app\main\default\sharingRules\CMS_Page__c.sharingRules-meta.xml
16. Push source to your org by running `sfdx force:source:push -u ScratchOrgName`
17. To publish PartnerWorld experience site run `sfdx force:community:publish --name PartnerWorld -u ScratchOrgName`
18. To publish MPW experience site run `sfdx force:community:publish --name MPW -u ScratchOrgName`
19. Open the scratch org by running `sfdx force:org:open -u ScratchOrgName`


### Reset Password for Scratch Org User
- To reset scratch org user's password, From Setup, Search for "Users" in Quick Find
- Click on the checkbox for the current user and select reset password
- Reset the password by clicking on the link sent to the linked org email address


---------------------------------------------------------------------------------------

# Setup and enable users to an experience site

After the deployment is successfully completed follow the steps to setup and enable users to an experience site.
Follow the same procedure for Dev org or a scratch org
## 1. Enable users for experience site 

- From Setup (gear icon - top right corner), enter Digital Experiences in the Quick Find box, then click Digital Experiences | Settings
- Enable **Allow using standard external profiles for self-registration, user creation, and login** under Role and User Settings. Click Save. (Click OK on the alert)
- From Setup, enter All Sites in the Quick Find box, then click All Sites under Digital Experiences. To access the site workspace, click Workspaces next to its name (PartnerWorld And MPW)
- Click Administration | Members
- To add members using permission sets:
  - Select the permission sets you want to allow access to your site. In our case it would be **CMS_MPW_Community**
  - Click Add
- Click Save

## 2. Assigning role to the logged in user

We need to assign role to logged in user, which would be Enabling the Customer as User to experience site

- From Setup, in the Quick Find box, enter Users, then select Users
- Click on edit next to logged in user name
- Click role dropdwon and select **EDGE Custom Role**
- Click Save

## 3. Enable Customer User for a contact

Next step is to Enable Contact by setting **Enable Customer User** for the contact

- From the App launcher, search for 'Accounts' and select Accounts under Items
- From the Accounts tab, Select the 'Hilton Union Square' account.
- On the account record page that opens, In the Related tab under the 'Contacts' section , click on the contact 'Pat Stumuller'.
- On the contact page, Click on the dropdown icon (After the 'New Note' button) on the utility bar (top-right corner) and select 'Enable User Customer'
- Click Save

## 4. Add Permission set for user

Next, for this same user which was created we need to assign **CMS_MPW_Community** permission set

- Scroll down to 'Permission Set Assignments' and click on 'Edit Assignments'
- Add the permission set **CMS_MPW_Community** to the Permission Set Assignments related list. (Click OK on the alert)

## 5. Generate Password for user

Now we need to Generate new password for newly created user

- From Setup, in the Quick Find box, enter Users, then select Users
- Select the checkbox next to the user's name
- Click Reset Password. The user (Pat Stumuller on the corresponding email ID) receives an email that contains a link and instructions to reset the password
- After password reset is successful, user can login to experience site

## 6. Login to experience site

- From Setup, in the Quick Find box search for 'All Sites' and click on it.
- Copy the URL and paste it in a new private window
- Login with the username and password of the user
- The PartnerWorld app will now be visible, Click on 'Demo Page' or 'Demo Page' on the navigation bar to view it.


# For enabling Enhanced Approval Process

## 1. Activate Approval Process
- From Setup, in the Quick Find search for 'Approval Processes' and click on it.
- Select 'CMS Page' in Manage Approval Processes For dropdown
- Activate 'Enhanced Approval Process'.
- Deactive any other active approval process.

## 2. Active Flows
- From Setup, in the Quick Find search for 'Approval Processes' and click on it.
- Click on 'Create Or Update Section' and on the new page click on 'Activate' on the right side.
- Follow same procedure for 'Create Or Update Tile', 'Create Or Update Link' and 'Email Business Controls'.

## Experience Site Public Access For Guest Users
Please refer to the following document to know more about how to enable site level access for guest users, navigation menu access and configuring Page Level access.
https://ibm.ent.box.com/notes/985481805935

## Future Enhancements
If new objects are added or any of the existing objects are modified to include more attributes or new classes are added, please remember to update the necessary permissions for the objects/fields/classes in the appropriate permission sets and the guest user profiles for the experience sites.

# Dev best practices/expected actions for Raising PR
- Please refer to checklist here https://ibm.ent.box.com/notes/985412026996 and follow the guidelines
