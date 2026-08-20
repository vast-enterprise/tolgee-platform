# Progress

- Date: 2026-08-18
- Branch: main
- Changes: Added a DEV K8S deployment template for the existing Tolgee image, persistent `/data` storage, ConfigMap input, and deployment through the existing ALB at `tolgee.vast-internal.com` in `env-base`.
- Verification: `bash -n scripts/deploy-dev.sh` and `git diff --check` passed. Gradle task verification was blocked because no Java Runtime is installed. Automated deployment was not run because it requires company AWS credentials, kubeconfig, and cluster access.
- Remaining issues: ECR publication is blocked by proxy interruptions while fetching Docker Hub layers. The deployment now uses the official multi-architecture `tolgee/tolgee:latest` image directly; rollout is pending.

- Date: 2026-08-19
- Branch: main
- Changes: Switched the Tolgee Ingress from the unresolvable `vast-internal.com` hostname/ALB to the existing DEV wildcard domain `tolgee.devops.tripo3d.ai` and shared NGINX Ingress. No Deployment, Service, PVC, or other workload was changed.
- Verification: DNS resolves `tolgee.devops.tripo3d.ai`; manifest change is limited to the Tolgee Ingress routing configuration. Final HTTPS check is pending.
- Remaining issues: None. HTTPS health check returned HTTP 200 with `status: UP`.

- Date: 2026-08-19
- Branch: main
- Changes: Deployed Tolgee to the DEV EKS cluster `tripo-dev-service`, namespace `env-base`, with one replica, its own `tolgee` Service, PVC, ConfigMap, and ALB Ingress for `tolgee.vast-internal.com`. Existing workloads were not modified.
- Verification: `kubectl rollout status deployment/tolgee` passed; Pod is `1/1 Running`, `0` restarts, and the container is using the amd64-compatible official multi-architecture image `tolgee/tolgee:latest`. ALB Ingress reconciliation succeeded. The application health endpoint passed the Pod readiness probe.
- Remaining issues: External HTTPS curl from this workstation returned a network/DNS connection error, so public hostname reachability still needs verification from the user's network. ECR push remains incomplete; the running Pod pulls directly from Docker Hub.

- Date: 2026-08-18
- Branch: current working branch
- Changes: Verified the local Tolgee Docker import flow for nested JSON and restored the original PostgreSQL 11 Compose data configuration. No database backup or restore was used for this verification.
- Verification: `docker compose ps` reports all services running and the Tolgee health endpoint returns HTTP 200. Uploaded the nested JSON fixture through the local UI to project 1; preview showed 10 translations, import returned HTTP 200, and the translations page showed 10 keys including the nested `page1.*` keys.
- Remaining issues: `project_id=2` does not exist in the current database; use `http://localhost:8091/projects/1/import`. The Compose file has no functional diff from its original configuration.

- Date: 2026-08-19
- Branch: feat/k8s
- Changes: Enabled Tolgee authentication and native username/password login in the DEV Kubernetes ConfigMap environment input. No password or OAuth secret was added to the repository.
- Verification: Applied the ConfigMap, restarted `deployment/tolgee`, rollout completed with one ready pod, ConfigMap values read back as `true`, `/data/initial.pwd` is present, and the HTTPS endpoint returned HTTP 200. Google OAuth remains unconfigured.
- Remaining issues: The existing `admin` account password is required for the new login page; retrieve it from `/data/initial.pwd` if it was generated automatically.

- Date: 2026-08-19
- Branch: feat/k8s
- Changes: Enabled public native-user registration for the DEV Tolgee deployment.
- Verification: Applied the ConfigMap, restarted `deployment/tolgee`, rollout completed successfully, all three authentication settings read back as `true`, and the HTTPS endpoint returned HTTP 200.
- Remaining issues: None known.

- Date: 2026-08-20
- Branch: feat/k8s
- Changes: Configured the Tolgee DEV ConfigMap for Feishu SMTP over SSL (`smtp.feishu.cn:465`) with `Tolgee <luonan@vastai3d.com>` as sender. Added Deployment references to the out-of-band `tolgee-smtp` Secret keys `username` and `password`; created the Secret with the supplied username and a password placeholder only.
- Verification: Applied the ConfigMap and manifest to `tripo-dev-service-cluster-edit`, namespace `env-base`; `deployment/tolgee` rolled out successfully, the ready Pod loaded the SMTP host and username from its configuration, its in-cluster health endpoint returned `UP`, and the public HTTPS health endpoint returned HTTP 200.
- Remaining issues: Replace the placeholder Secret password with the real Feishu SMTP/client app password and restart `deployment/tolgee`; invitation email delivery has not been tested because no real password was provided.

- Date: 2026-08-20
- Branch: feat/k8s
- Changes: Replaced the `env-base/tolgee-smtp` password placeholder through Kuboard and restarted the Tolgee Deployment so the updated Secret is injected into the Pod.
- Verification: `deployment/tolgee` rolled out successfully with one ready Pod; the container confirms that a non-placeholder SMTP password is loaded, and `https://tolgee.devops.tripo3d.ai/actuator/health` returned HTTP 200 with status `UP`.
- Remaining issues: Send a real project invitation to verify Feishu SMTP authentication and delivery end to end.
