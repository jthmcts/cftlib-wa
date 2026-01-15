# Work Allocation integration with cftlib
The aim of this project is to demonstrate how work allocation can be integrated with cftlib.

## Rationale
Because developing in the preview environment is :poop:.

## Approach
- The main obstacle with integrating work allocation is the dependency on Azure Service Bus.
- ASB is used to move data from one database to another, so we can simulate this by directly updating the database.
- The approach in this project is to use a scheduled task to move data `message_queue_candidates` to `wa_case_event_messages` table.
- For simplicity, the `wa_case_event_messages` table has been added to the `datastore` database.
- All other work allocation services are run in docker containers and scheduled tasks simulate the various cron jobs.

## Setup
1. Clone the Camunda definition repositories locally.
   - `https://github.com/hmcts/wa-standalone-task-bpmn`
   - `https://github.com/hmcts/prl-wa-task-configuration` (or your equivalent)

   Then set the following environment variables to point to the local paths of those repositories:
   - `WA_STANDALONE_TASK_REPO_PATH`
   - `WA_TASK_CONFIGURATION_REPO_PATH`

   e.g.
   - `export WA_STANDALONE_TASK_REPO_PATH=/Users/doris/wa/wa-standalone-task-bpmn`
   - `export WA_TASK_CONFIGURATION_REPO_PATH=/Users/doris/prl/prl-wa-task-configuration`

   See `/bin/import-camunda-definitions.sh` for more details.

   Ensure the following variables in the script are correct for your service:
   - `DEPLOYMENT_SOURCE="prl"`
   - `TENANT_ID="privatelaw"`

2. Add the following to your `bootWithCCD` in `build.gradle`. Ensure `DATA_STORE_S2S_AUTHORISED_SERVICES` is correct for your service.
   ```bash
   environment("RSE_LIB_ADDITIONAL_DATABASES", "camunda,wa_workflow_api,cft_task_db,cft_task_db_replica")
   environment("DATA_STORE_S2S_AUTHORISED_SERVICES", "ccd_data,ccd_gw,ccd_ps,jui_webapp,pui_webapp,xui_webapp,aac_manage_case_assignment,am_role_assignment_service,wa_task_configuration_api,wa_task_management_api,wa_workflow_api,wa_task_monitor,ccd_case_document_am_api,prl_cos_api,wa_case_event_handler,hmc_cft_hearing_service,fis_hmc_api,ccd_next_hearing_date_updater")
   environment("FEATURE_WORKALLOCATION_ENABLED", "true")
   environment("SERVICES_WORK_ALLOCATION_TASK_API", "http://host.docker.internal:9193")
   environment("HEALTH_WORK_ALLOCATION_TASK_API", "http://host.docker.internal:9193/health")
   ```
   
3. Start your bootWithCCD

4. Execute the following database update:
    ```sql
    update message_queue_candidates set published = now();
    ```
5. Run the `sql/wa-case-event-messages.sql` script in the datastore database.

6. Build this project:
   ```bash
   ./gradlew assemble
    ```
7. Run docker compose:
   ```bash
   docker compose up -d
   ```
8. Import your Camunda definition files
   ```bash
   ./bin/import-camunda-definitions.sh
   ```
