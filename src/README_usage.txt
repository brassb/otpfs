
To Mount: ./otpfs -f --user_data source_dir=/path/to/raw/storage /path/to/otpfs_mount_point

  NOTE: The argument for --user_data is a comma-delimited list of name-value pairs.
  Additional name-value pairs include the following:

    serial_type=int|long      (default is int)
    serial_start=0            (default is 0)
    serial_step=1             (default is 1)
    show_passphrase=no|yes    (default is no)

