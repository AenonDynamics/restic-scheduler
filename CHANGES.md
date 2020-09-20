### 0.6.1 ###

* Bugfix: for compatibility with legacy restic version `<0.9.6` without `RESTIC_CACHE_DIR` environment variable support, the `--cache-dir` argument is set to `RESTIC_CACHE_DIR`

### 0.6.0 ###

* Changed: restic cache dir is provided as environment variable (can be overridden by env file or additional argument)

### 0.5.0 ###

* Added: systemd units to send backup logs (from journald) via [taskrunner](https://github.com/AenonDynamics/taskrunner-sh) to custom http endpoint

### 0.4.0 ###

* initial public release