# terraform-gcp

Terraform code that deploys a small, locked-down web server on Google Cloud Platform. I wrote it as a take-home assignment for a company's internship application.

## What it does
- Creates a Debian 12 VM (`e2-micro`) in `europe-north1`.
- Restricts SSH to Identity-Aware Proxy (IAP): port 22 is open only to Google's IAP range `35.235.240.0/20`, so the VM has no public SSH.
- Opens HTTP/HTTPS (80/443) to the internet for the web server.
- Gives the VM's service account only the scopes it needs (`logging-write`, `monitoring-write`).
- Runs a startup script that installs Nginx, serves a simple page and gets a Let's Encrypt certificate with Certbot (HTTP redirects to HTTPS).
- Creates a Cloud DNS A record pointing a domain at the VM's external IP.

## Tech stack
Terraform (Google provider ~> 5.0), GCP Compute Engine, VPC firewall rules, IAP, Cloud DNS, Nginx, Certbot, Bash.

## Files
| File | Purpose |
|---|---|
| `main.tf` | VM, firewall rules, DNS record (plus a commented-out IAM binding for IAP access) |
| `variables.tf` | Project, region, zone, instance name, machine type |
| `outputs.tf` | External IP and URLs |
| `startup-script.sh` | Installs Nginx and Certbot on first boot |

## How to run
Requirements: Terraform >= 1.0, an authenticated Google Cloud SDK, the IAP API enabled, and the `roles/iap.tunnelResourceAccessor` role for your user.

```bash
echo 'project_id = "<your-gcp-project-id>"' > terraform.tfvars
terraform init
terraform plan
terraform apply
terraform output
```

The DNS zone and domain in `main.tf`, `outputs.tf` and `startup-script.sh` point to the sandbox domain I was given. Change them to a Cloud DNS zone you control.

Connect over IAP:
```bash
gcloud compute ssh "$(terraform output -raw vm_name)" \
  --zone=europe-north1-a --tunnel-through-iap --project=<your-gcp-project-id>
```

## What I learned
- Writing infrastructure as code with variables and outputs, and working through `plan`/`apply`.
- Removing public SSH by tunnelling through IAP, and scoping firewall rules with network tags.
- Automating server setup and TLS with a startup script.
- Working with IAM in a shared project: the IAP role binding is left commented out because an administrator granted that access.
