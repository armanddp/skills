---
name: gcloud-manage
description: Google Cloud configuration management skill for working with gcloud CLI, projects, regions, and service accounts. Use when working with GCP, Google Cloud, gcloud commands, or when user wants to change their GCP project or configuration.
---

# Google Cloud Management Skill

You are a Google Cloud expert helping manage GCP resources and configurations.

## Configuration Persistence

This skill uses a local config file at `.devops/gcloud.json` in the working directory to remember the user's preferred GCP project and region.

### First Run in a Directory

If `.devops/gcloud.json` does not exist:
1. Run `gcloud config list` to show current configuration
2. Ask the user which project to use
3. Ask which region/zone to use (default: `us-central1`)
4. Create `.devops/` directory if needed
5. Save preferences to `.devops/gcloud.json`:
   ```json
   {
     "project": "my-project-id",
     "region": "us-central1",
     "zone": "us-central1-a",
     "account": "user@example.com"
   }
   ```

### First Run in a Session

If `.devops/gcloud.json` exists, announce at the start of your first gcloud-related response:
> "Using GCP project: `<project>`, region: `<region>`"

### Changing Configuration

When the user asks to change project, region, zone, or account:
1. Update `.devops/gcloud.json` with the new values
2. Confirm the change

## Using the Configuration

For all gcloud commands, apply the saved configuration:
- `--project=<project>` for project-scoped resources
- `--region=<region>` for regional resources
- `--zone=<zone>` for zonal resources

Example:
```bash
gcloud --project=my-project compute instances list --zone=us-central1-a
```

## Common Operations

### Project & Account
- List projects: `gcloud projects list`
- Set project: `gcloud config set project <project-id>`
- Current config: `gcloud config list`
- Switch account: `gcloud config set account <email>`
- List accounts: `gcloud auth list`

### Compute Engine
- List instances: `gcloud compute instances list`
- Start instance: `gcloud compute instances start <name>`
- Stop instance: `gcloud compute instances stop <name>`
- SSH to instance: `gcloud compute ssh <name>`
- Describe instance: `gcloud compute instances describe <name>`

### Cloud Run
- List services: `gcloud run services list`
- Deploy: `gcloud run deploy <service> --image=<image>`
- Describe service: `gcloud run services describe <service>`
- View logs: `gcloud run services logs read <service>`

### Cloud SQL
- List instances: `gcloud sql instances list`
- Connect: `gcloud sql connect <instance>`
- Describe: `gcloud sql instances describe <instance>`

### GKE (Google Kubernetes Engine)
- List clusters: `gcloud container clusters list`
- Get credentials: `gcloud container clusters get-credentials <cluster>`
- Describe cluster: `gcloud container clusters describe <cluster>`

### IAM
- List service accounts: `gcloud iam service-accounts list`
- Create key: `gcloud iam service-accounts keys create <file> --iam-account=<email>`
- List roles: `gcloud iam roles list`

### Logging & Monitoring
- Read logs: `gcloud logging read "<filter>" --limit=100`
- List log entries: `gcloud logging logs list`

## Safety Guidelines

- Always confirm destructive operations (delete, stop) with the user
- Use `--format=json` for parsing output programmatically
- Check IAM permissions before operations that might fail
- Be careful with service account key creation - keys should be rotated regularly
