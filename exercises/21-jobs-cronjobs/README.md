# Exercise 21 — Jobs and CronJobs

**Domain:** Workloads & Scheduling (15%)

**Difficulty:** Medium | **Time:** 15 min

---

## Context

Jobs run one or more pods to completion. Unlike Deployments (which keep pods running indefinitely), Jobs don't restart failed pods by default — they just retry. CronJobs schedule Jobs to run on a schedule, like cron in Linux.

On the exam, you'll create both and understand backoff limits, completions, and restart policies. The questions are straightforward: create a job that runs a script to completion, handle failures, maybe schedule it for later.

---

## Tasks

1. Create a Job named `cleanup-job` that runs the image `busybox:1.37` with command `echo "Cleanup complete"`. Set it to run 3 times (completions=3) serially (parallelism=1).

2. Verify the Job runs to completion (status should show 3/3). Check the pod logs.

3. Create a CronJob named `hourly-backup` that runs `backup-script.sh` every hour. Use the same busybox image. Let it keep 3 successful job histories (successfulJobsHistoryLimit=3).

4. List CronJobs and check the next scheduled time.

5. Manually trigger the CronJob once (create a Job from it).

6. Verify the manual Job ran and logged output.

---

## Hints

- **completions=3**: The Job must run 3 times successfully before completion.
- **parallelism=1**: Run one pod at a time, serially.
- **backoffLimit=3**: Retry failed pods up to 3 times, then give up.
- **historyLimit**: Keep this many job history records. Default is 3 for success and 1 for failure.
- For CronJobs, use `spec.schedule` with standard cron syntax (e.g., `0 * * * *` = every hour).
- `kubectl get cronjob` shows the next scheduled time in the LAST SCHEDULE column.
- To manually trigger a CronJob: `kubectl create job <jobname> --from=cronjob/<cronname>`.

---

## What tripped me up

I expected completions=3 to mean "run 3 pods in parallel." It doesn't — it means "successfully complete 3 times total." With parallelism=1, those 3 runs are serial. If parallelism=3 and completions=3, all 3 run together.

I also forgot that CronJobs are in the `batch` API group, not `v1`. The YAML syntax is slightly different from Jobs.

When a Job fails, the pod isn't deleted unless the Job's `ttlSecondsAfterFinished` is set. Failed pods stick around for debugging, which is good. But if you're running a test Job, clean it up afterward or you'll have orphan pods.

---

## Verify

```bash
# Check Job completion
k get job cleanup-job -o wide
k get pod -l job-name=cleanup-job

# Check logs
k logs -l job-name=cleanup-job --all-containers=true

# Check CronJob
k get cronjob hourly-backup

# Check if CronJob created a Job
k get job -l cronjob-name=hourly-backup

# Check triggered Job
k logs -l job-name=<triggered-job-name> --all-containers=true
```

---

## Cleanup

```bash
k delete job cleanup-job
k delete cronjob hourly-backup
k delete job --selector cronjob-name=hourly-backup
```

---

<details>
<summary><b>Solution</b></summary>

**Step 1: Create the Job**

```bash
k create job cleanup-job \
  --image=busybox:1.37 \
  --command -- echo "Cleanup complete" 

# This creates a Job that runs once. For 3 completions:
k patch job cleanup-job --type json -p '[
  {"op": "replace", "path": "/spec/completions", "value": 3},
  {"op": "replace", "path": "/spec/parallelism", "value": 1}
]'
```

Or using a declarative YAML:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: cleanup-job
spec:
  completions: 3
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: cleanup
        image: busybox:1.37
        command: ["echo", "Cleanup complete"]
      restartPolicy: Never
```

**Step 2: Verify Job runs**

```bash
k get job cleanup-job -w
# Watch until Completions shows 3/3

k get pod -l job-name=cleanup-job
# Should show 3 pods

k logs job/cleanup-job --all-containers=true
# Shows all pod logs from the Job
```

**Step 3: Create CronJob**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hourly-backup
spec:
  schedule: "0 * * * *"  # Every hour at :00
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: busybox:1.37
            command: ["echo", "Backup at $(date)"]
          restartPolicy: OnFailure
```

```bash
k apply -f cronjob.yaml
```

**Step 4: List CronJobs**

```bash
k get cronjob hourly-backup
# SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
# 0 * * * *     False     0        <none>          20s
```

**Step 5: Manually trigger**

```bash
k create job hourly-backup-manual-1 --from=cronjob/hourly-backup
```

**Step 6: Verify manual Job**

```bash
k get job -l cronjob-name=hourly-backup
k logs job/hourly-backup-manual-1
```

</details>

---

## Resources

- [Kubernetes Batch Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- [Job backoff restart policy](https://kubernetes.io/docs/concepts/workloads/controllers/job/#pod-backoff-failure-policy)
