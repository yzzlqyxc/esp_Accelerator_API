## Starting Jobs Serially

In `soft/common/drivers/linux/libesp/libesp.c`, when starting a job to the accelerator in the `esp_run_parallel` function, instead of calling `accelerator_thread_serial` in a pthread, we now invoke this function directly.

## Waiting for Jobs Serially

In `soft/common/drivers/linux/esp/esp.c`, the function `esp_access_ioctl` normally calls `esp_wait` after submitting a job to block until completion. To avoid blocking subsequent jobs, we no longer wait immediately after each submission.  

Instead of using `pthread_join` in `accelerator_thread_serial`, I used a new function called `accelerator_thread_wait`, which waits for all jobs to complete serially. This function eventually calls the `esp_wait_ioctl` function in `esp.c`, which invokes `esp_wait`.

## Concerns

At the end of `esp_access_ioctl`, the function calls `esp_update_status`. Since starting a job and waiting are now separated, this may affect how the status is updated. The current version of `esp_update_status` may not reflecting the correct state after these changes.  

Additionally, I have not added lock acquisition or release during job waiting, and I am unsure whether synchronization is required here.