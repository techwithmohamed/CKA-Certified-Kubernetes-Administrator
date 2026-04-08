# Mock Exams — CKA Practice Tests

Two comprehensive practice exams that simulate real CKA exam conditions.

## How to Use

1. **Read ONLY the question file** (MOCK-EXAM-01.md or MOCK-EXAM-02.md)
2. **Set a 2-hour timer** — Do not exceed this time
3. **Solve each question** in your own Kubernetes cluster or practice environment
4. **Keep score** — Track how many questions you complete correctly
5. **Do NOT look at solutions** until after you finish
6. **Review solutions** — Compare your answers against the solution file
7. **Score yourself** — 10+/15 correct = passing (66%), 12+/15 = strong, 15/15 = ready for real exam

## Mock Exam 01

**Focus:** Troubleshooting (30%) and Cluster Architecture (25%) domains

- 15 questions covering all CKA domains
- 2-hour time limit
- Questions emphasize Pod lifecycle, RBAC, Node management, Persistent Storage, NetworkPolicy, Helm, Containers, HPA, Ingress, Kubeadm, API Server, StatefulSets, Resource Quotas, Pod Security, and Multi-container Pods

**Files:**
- Questions: [MOCK-EXAM-01.md](MOCK-EXAM-01.md)
- Solutions: [MOCK-EXAM-01-SOLUTIONS.md](MOCK-EXAM-01-SOLUTIONS.md)

## Mock Exam 02

**Focus:** Services & Networking (20%) and Workloads (15%) domains

- 15 questions covering all CKA domains with different scenarios than Exam 01
- 2-hour time limit
- Questions emphasize Service DNS, Deployment updates, Secrets, DaemonSets, API Server debugging, ConfigMap updates, PriorityClass, Service types, CronJobs, Certificate rotation, Pod Disruption Budgets, CRDs, Pod Security Policies, StorageClass, and Resource governance

**Files:**
- Questions: [MOCK-EXAM-02.md](MOCK-EXAM-02.md)
- Solutions: [MOCK-EXAM-02-SOLUTIONS.md](MOCK-EXAM-02-SOLUTIONS.md)

## Study Recommendations

**Timing in your exam prep:**

1. Complete all 22 exercises in the `exercises/` directory first
2. Take Mock Exam 01 — identify weak domains
3. Review the corresponding exercises for topics you missed
4. Take Mock Exam 02 — validate improvement
5. If both scores are 12+/15, you're ready for the real CKA exam
6. If lower, review the solutions and spend more time on the weaker domains

**Real exam information:**

- Real CKA exam has 15-17 questions in 2 hours
- These mock exams have 15 questions in 2 hours (realistic pacing)
- Real exam allows access to kubernetes.io/docs during the exam
- Questions are hands-on, terminal-based (like these mocks)
- Passing score is 66% (10+/15 correct)

## Tips for Mock Exam Success

1. **Use imperative commands** — Not YAML from memory
2. **Always verify your work** — Use `kubectl get`, `describe`, `logs` after each task
3. **Call kubectl without alias** — On real exam, only `k` alias works
4. **Time management** — Allocate 7-8 minutes per question average
5. **Read questions carefully** — Misunderstanding costs more time than solving
6. **Flag and come back** — If a question takes >10 minutes, flag it and tackle easy ones first
7. **Document nothing** — The real exam doesn't allow copy-paste to external tools, so don't practice with tricks
8. **Test your solutions** — Pod should be running, service should be accessible, etc. Don't just create and move on

## Not Real Exam Questions

These are practice scenarios inspired by real CKA exam domains and topics. They are independently written training materials designed to help you prepare. They are NOT actual CKA exam questions. The Linux Foundation does not share real exam questions publicly. For more information, see [CONTRIBUTING.md](../CONTRIBUTING.md) and [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md).

---

After completing both mock exams successfully, you're ready for killer.sh (harder practice) or the real CKA exam.
