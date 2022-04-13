# [GSP349 Deploy and Manage Apigee X: Challenge Lab](https://www.cloudskillsboost.google/focuses/20940?parent=catalog)

## Task 1. Modify the environments and environment groups
- Open the [Apigee Console](https://apigee.google.com).
#### Add staging environment
- Navigate to ***Admin > Environments > Overview***.
- In the upper right corner, click ***+Environment***.
- Specify ***[ staging ]*** for Display name and Environment name. Other fields should remain unchanged.
- Click ***Create***.
#### Add staging environment group
- Navigate to ***Admin > Environments > Groups***.
- In the upper right corner, click ***+Environment Group***.
- Name the environment group ***[ staging-group ]***, and then click ***Add***.
- In the ***[ staging-group ]*** environment group box, click the ***edit*** (pencil) button (pencil icon).
- In the ***Environments*** box, click the ***+ button***.
- Select the ***[ staging ]*** environment, and then click ***Add***.
- Leave the default hostname that is created for the new environment group.

## Task 2. Use the provisioning wizard to set up access routing
- Open the [Apigee Console Setup](https://apigee.google.com/setup).
- From the lab information on the left, copy your ***[ Google Cloud Project ID ]*** into the clipboard.
- Paste the ***[ Google Cloud Project ID ]*** into the Project text box.
- Click ***Continue***.
- Wait for provisioning to complete.
#### Start monitoring script
- Return to the Cloud Console tab.
- On the top-right toolbar, click the ***Activate Cloud Shell*** button.
- In the Cloud Shell, verify the variable with your Apigee org name.
  - `echo ${GOOGLE_CLOUD_PROJECT}`
  - `export INSTANCE_NAME=eval-instance; export ENV_NAME=eval; export PREV_INSTANCE_STATE=; echo "waiting for runtime instance ${INSTANCE_NAME} to be active"; while : ; do export INSTANCE_STATE=$(curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" -X GET "https://apigee.googleapis.com/v1/organizations/${GOOGLE_CLOUD_PROJECT}/instances/${INSTANCE_NAME}" | jq "select(.state != null) | .state" --raw-output); [[ "${INSTANCE_STATE}" == "${PREV_INSTANCE_STATE}" ]] || (echo; echo "INSTANCE_STATE=${INSTANCE_STATE}"); export PREV_INSTANCE_STATE=${INSTANCE_STATE}; [[ "${INSTANCE_STATE}" != "ACTIVE" ]] || break; echo -n "."; sleep 5; done; echo; echo "instance created, waiting for environment ${ENV_NAME} to be attached to instance"; while : ; do export ATTACHMENT_DONE=$(curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" -X GET "https://apigee.googleapis.com/v1/organizations/${GOOGLE_CLOUD_PROJECT}/instances/${INSTANCE_NAME}/attachments" | jq "select(.attachments != null) | .attachments[] | select(.environment == \"${ENV_NAME}\") | .environment" --join-output); [[ "${ATTACHMENT_DONE}" != "${ENV_NAME}" ]] || break; echo -n "."; sleep 5; done; echo; echo "${ENV_NAME} environment attached"; echo "***ORG IS READY TO USE***";`
- Wait for the provisioning Apigee evaluation organization to complete.
#### Set up access routing
- Next to Access routing, click ***Edit***.
- Select ***Enable internet access***.
- Select ***Use wildcard DNS service***.
- For Subnetwork, select ***[ api-subnet ]***.
- Click ***Set Access***.

# Task 3. Create and activate a NAT address for the instance
##### Create NAT address
- Open the [Apigee API](https://cloud.google.com/apigee/docs/reference/apis/apigee/rest).
- find ***organizations.instances.natAddresses***.
- Under ***organizations.instances.natAddresses***, click ***create***.
- In the Try this API pane, set the parent to: `organizations/{YOUR PROJECTID}/instances/eval-instance`
- Change `{YOUR PROJECTID}` to your ***[ Google Cloud Project ID ]***.
- Click ***Add request body parameters, and then click name***.
- Between the double quotes, set the string to: ***[ apigee-nat-ip ]***.
- Click ***Execute***.
#### Activate NAT address
- In the left pane, under ***organizations.instances.natAddresses***, click ***activate***.
- In the Try this API pane, set the parent to: `organizations/{YOUR PROJECTID}/instances/eval-instance/natAddresses/apigee-nat-ip`
- Change `{YOUR PROJECTID}` to your ***[ Google Cloud Project ID ]***.
- Click ***Execute***.

# Task 4. Create a Cloud Armor security policy and attach it to the global load balancer
#### Create the Cloud Armor policy
- In the Cloud Console tab, on the Navigation menu (navigation menu button), navigate to ***Network security > Cloud Armor***.
- Click ***Create policy***.
- For ***Name***, specify ***[ protect-apigee ]***.
- ***Default rule action choose Allow***.
- Click ***Next step > + Add rule > Advanced mode***.
- For ***Match***, specify the following expression: `evaluatePreconfiguredExpr('rce-stable')`.
- Leave Action set to Deny, and leave Deny status set to 403 (Forbidden).
- Set ***Priority*** to ***1000***.
- Click ***Done***.
- Click ***Create policy***.
#### Attach the Cloud Armor policy
- Next to ***protect-apigee***, click the policy menu button (three dots), and then click ***Apply policy to target***.
- Click ***+ Add target***.
- For the Target dropdown, select ***apigee-proxy-backend***, and then click ***Add***.

# Task 5. Attach the staging environment to the runtime instance
#### Attach the environment to the runtime instance
- Confirm that the commands run in Cloud Shell returned `***ORG IS READY TO USE***`.
- To begin the process of attaching the staging environment to the runtime, run the following command:
`export INSTANCE_NAME=eval-instance; curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" -H "Content-Type: application/json" -X POST "https://apigee.googleapis.com/v1/organizations/${GOOGLE_CLOUD_PROJECT}/instances/${INSTANCE_NAME}/attachments" -d '{ "environment": "staging" }' | jq`
- Check the status of the staging attachment using this command:
`export ATTACHING_ENV=staging; export INSTANCE_NAME=eval-instance; echo "waiting for ${ATTACHING_ENV} attachment"; while : ; do export ATTACHMENT_DONE=$(curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" -X GET "https://apigee.googleapis.com/v1/organizations/${GOOGLE_CLOUD_PROJECT}/instances/${INSTANCE_NAME}/attachments" | jq "select(.attachments != null) | .attachments[] | select(.environment == \"${ATTACHING_ENV}\") | .environment" --join-output); [[ "${ATTACHMENT_DONE}" != "${ATTACHING_ENV}" ]] || break; echo -n "."; sleep 5; done; echo; echo "***${ATTACHING_ENV} ENVIRONMENT ATTACHED***";`
- When the text `***staging ENVIRONMENT ATTACHED***` is printed,***You have completed this challenge***.
