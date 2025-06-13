# dbt Snowplow Unified Keyhole Backfill Instructions

This guide provides steps to perform a keyhole backfill for the Snowplow Unified package. This reposistory is a fork of Snowplow Unified and allows you to simply update your packages.yml file to run a keyhole backfill.

**Warning:**  
This process involves editing data in production. It is critical to carefully test all changes in a development environment and strictly follow the instructions provided to avoid unintended consequences.

## How it works

The keyhole backfill works by doing the following when `snowplow__keyhole_backfill_enabled` is set to true:
- Disabling the post-hooks which update the Snowplow Incremental Manifest - this prevents your incremental manifest from losing the current state of your models.
- Setting the base event run dates to be the keyhole dates that are provided.
- Overriding the `get_run_limits` macro to provide information to the console for when the keyhole backfill is running
- Removing the logic in the base events table to fetch all events for sessions that occur within the keyhole period.

## Steps

The keyhole backfill process has been simplified to use pre-defined macros and configuration variables. Follow these steps to set up and execute the backfill:

1. **Update packages.ymls**
    - Update your `packages.yml` to the following and replace `snowplow_unified` (and remove `snowplow_utils` if you have specified it):
    ```
    packages:
    - git: "https://github.com/snowplow-industry-solutions/dbt-unified-keyhole-backfill"
      revision: "main"
    ```

3. **Configure Variables**  
    - Update your `dbt_project.yml` file to include the following variables:
      ```yaml
      vars:
         snowplow__keyhole_backfill_enabled: true
         snowplow__keyhole_backfill_lower_limit: '2025-03-10 00:00:00'
         snowplow__keyhole_backfill_upper_limit: '2025-03-20 00:00:00'
      ```
      - Set `snowplow__keyhole_backfill_enabled` to `true` to activate the backfill process.
      - Define the `lower_limit` and `upper_limit` for the backfill using the appropriate date range.
      - If you are unable to process all the data within your backfill limit range due to data processing limitations, you will need to manually update and set the dates for each run.

4. **Run the Model**  
    - Execute the dbt model as usual (`dbt run` or `dbt run --select snowplow_unified`). The macros will automatically apply the keyhole backfill logic based on the configured variables.

5. **Verify Reprocessed Data**  
    - Check the `snowplow_unified_events_this_run` table to confirm the correct data was reprocessed by validating the `max` and `min` dates.

## Notes
- Ensure all changes are reviewed and tested in a development environment before applying them to production.
- The macros are designed to minimize manual edits and streamline the backfill process.
- Disable the backfill by setting `snowplow__keyhole_backfill_enabled` to `false` once the process is complete.
- If you run `dbt deps` you will need to copy across the macros again.
