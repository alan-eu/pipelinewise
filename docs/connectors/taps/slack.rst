
.. _tap-slack:

Tap Slack
----------

Configure your Slack workspace
''''''''''''''''''''''''''''''

The tap requires a `Slack Bot user token <https://api.slack.com/authentication/token-types#granular_bot>`_
to interact with your Slack workspace. You can obtain a token for a single workspace by creating a new
`Slack App <https://api.slack.com/apps?new_app=1>`_ in your workspace and assigning it the relevant
`scopes <https://api.slack.com/docs/oauth-scopes>`_.

As of right now, the minimum required scopes for this App are:
 - ``channels:history``
 - ``channels:join``
 - ``channels:read``
 - ``files:read``
 - ``groups:read``
 - ``links:read``
 - ``reactions:read``
 - ``remote_files:read``
 - ``remote_files:write``
 - ``team:read``
 - ``usergroups:read``
 - ``users.profile:read``
 - ``users:read``
 - ``users:read.email`` - This scope is only required if you want to extract the user emails as well.

Configuring what to extract
'''''''''''''''''''''''''''

PipelineWise configures every tap with a common structured YAML file format.
A sample YAML for Jira replication can be generated into a project directory by
following the steps in the :ref:`generating_pipelines` section.

Example YAML for ``tap-slack``:

.. code-block:: yaml

    ---

    # ------------------------------------------------------------------------------
    # General Properties
    # ------------------------------------------------------------------------------
    id: "slack"                            # Unique identifier of the tap
    name: "Slack"                          # Name of the tap
    type: "tap-slack"                      # !! THIS SHOULD NOT CHANGE !!
    owner: "somebody@foo.com"              # Data owner to contact
    #send_alert: False                     # Optional: Disable all configured alerts on this tap


    # ------------------------------------------------------------------------------
    # Source (Tap) - Github connection details
    # ------------------------------------------------------------------------------
    db_conn:
      token: "<SLACK_TOKEN>"                    # Slack API token
      start_date: "2020-09-01"                  # Start date. Data will be synced incrementally starting from this data

      #channels: ["ID1", "ID2", "ID3"]          # Optional: By default, the tap will sync all channels it has been invited to.
                                                # However, you can limit the tap to sync only the channels you specify by
                                                # adding their IDs to the config
      #exclude_archived: "false"                # Optional: You can control whether or not the tap will sync archived channels
                                                # by including the following in the tap config
      #private_channels: "false"                # Optional:, you can also specify whether you want to sync private
                                                # channels. By default private channels not synced
      #join_public_channels: "false"            # Optional: Auto-join every public channel.
                                                # If you do not elect to have the tap join all public channels you must
                                                # invite the bot to all channels you wish to sync.
      #date_window_size: "5"                    # Optional: Due to the potentially high volume of data when syncing certain streams
                                                # (messages, files, threads) this tap implements date windowing based on
                                                # a configuration parameter.
      #lookback_window: 14                      # Optional: Number of days to look back before the incremental start date.
                                                # This is useful to get all data from child streams that can't be extracted
                                                # incrementally, for example the message threads (conversation.replies method).
                                                # Default is 14 days.

    # ------------------------------------------------------------------------------
    # Destination (Target) - Target properties
    # Connection details should be in the relevant target YAML file
    # ------------------------------------------------------------------------------
    target: "snowflake"                       # ID of the target connector where the data will be loaded
    batch_size_rows: 20000                    # Batch size for the stream to optimise load performance
    stream_buffer_size: 0                     # In-memory buffer size (MB) between taps and targets for asynchronous data pipes
    default_target_schema: "slack"            # Target schema where the data will be loaded
    #default_target_schema_select_permission:  # Optional: Grant SELECT on schema and tables that created
    #  - grp_power
    #batch_wait_limit_seconds: 3600            # Optional: Maximum time to wait for `batch_size_rows`. Available only for snowflake target.

    # Options only for Snowflake target
    #archive_load_files: False                      # Optional: when enabled, the files loaded to Snowflake will also be stored in `archive_load_files_s3_bucket`
    #archive_load_files_s3_prefix: "archive"        # Optional: When `archive_load_files` is enabled, the archived files will be placed in the archive S3 bucket under this prefix.
    #archive_load_files_s3_bucket: "<BUCKET_NAME>"  # Optional: When `archive_load_files` is enabled, the archived files will be placed in this bucket. (Default: the value of `s3_bucket` in target snowflake YAML)


    # ------------------------------------------------------------------------------
    # Source to target Schema mapping
    # ------------------------------------------------------------------------------
    schemas:

      - source_schema: "slack"              # This is mandatory, but can be anything in this tap type
        target_schema: "slack"              # Target schema in the destination Data Warehouse
        target_schema_select_permissions:   # Optional: Grant SELECT on schema and tables that created
          - grp_stats

        # List of Slack tables to load into destination Data Warehouse
        # Tap-Slack will use the best incremental strategies automatically to replicate data
        tables:
          # Supported tables
          - table_name: "channels"
          - table_name: "users"
          - table_name: "channel_members"
          - table_name: "messages"
          - table_name: "threads"
          - table_name: "user_groups"
          - table_name: "teams"

          # Additional supported tables
          #- table_name: "files"
          #- table_name: "remote_files"

            # OPTIONAL: Load time transformations - you can add it to any table
            #transformations:
            #  - column: "some_column_to_transform" # Column to transform
            #    type: "SET-NULL"                   # Transformation type
