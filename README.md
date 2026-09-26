# d — Ansible Docker Deployment Pipeline

An Ansible-based provisioning and deployment pipeline that installs Docker on
target hosts and deploys a containerized web app via Docker Compose, with
role tests (Molecule), lint checks, and CI/CD (GitHub Actions + Jenkins).

Originally a one-task "Hello World" playbook — rebuilt into a small but
realistic DevOps project: idempotent roles, environment-specific inventories,
automated testing, and an optional Kubernetes manifest for the same app.

## What this demonstrates

- **Ansible**: roles, handlers, variables, templating, idempotency
- **Docker**: installing Docker Engine + Compose, deploying containers
- **Kubernetes**: equivalent Deployment/Service manifest (`k8s/`)
- **CI/CD**: GitHub Actions workflow (lint + Molecule tests) and a Jenkinsfile
- **Testing**: Molecule with the Docker driver

## Structure

```
d/
├── ansible.cfg
├── requirements.yml            # role/collection dependencies
├── site.yml                    # top-level playbook
├── inventories/
│   ├── staging/hosts.yml
│   └── production/hosts.yml
├── group_vars/all.yml
├── roles/
│   ├── docker/                 # installs Docker Engine + Compose plugin
│   │   ├── tasks/main.yml
│   │   ├── handlers/main.yml
│   │   ├── defaults/main.yml
│   │   └── molecule/default/   # Molecule test scenario
│   └── webapp_deploy/          # deploys the app via docker-compose
│       ├── tasks/main.yml
│       ├── templates/docker-compose.yml.j2
│       ├── defaults/main.yml
│       └── handlers/main.yml
├── k8s/
│   └── deployment.yaml         # same app, as a K8s Deployment + Service
├── Jenkinsfile
├── .github/workflows/ci.yml
└── .yamllint
```

## Usage

Install dependencies:

```bash
pip install ansible molecule molecule-plugins[docker] ansible-lint yamllint
ansible-galaxy install -r requirements.yml
```

Check syntax and lint:

```bash
yamllint .
ansible-lint
```

Run the role tests (spins up a throwaway Docker container as the "host"):

```bash
cd roles/docker
molecule test
```

Provision + deploy to staging:

```bash
ansible-playbook -i inventories/staging/hosts.yml site.yml --check   # dry run
ansible-playbook -i inventories/staging/hosts.yml site.yml
```

Provision + deploy to production (limits to hosts tagged `web`):

```bash
ansible-playbook -i inventories/production/hosts.yml site.yml --limit web
```

Deploy the same app to Kubernetes instead:

```bash
kubectl apply -f k8s/deployment.yaml
```

## CI/CD

- **GitHub Actions** (`.github/workflows/ci.yml`) runs `yamllint`,
  `ansible-lint`, and `molecule test` on every push/PR.
- **Jenkinsfile** mirrors the same pipeline (lint → test → deploy-to-staging
  on `main`) for teams running Jenkins instead of/alongside GitHub Actions.

## Notes

- Hosts in the inventories are placeholders — replace the IPs/hostnames with
  real targets (e.g. AWS EC2 instances) before running against anything but
  Molecule's local Docker test container.
- `webapp_deploy` defaults to deploying `nginxdemos/hello` on port 8080 as a
  stand-in; swap `webapp_image` in `group_vars/all.yml` for your own image.
