## Dependecies that we will use 
```
npm install axios bootstrap react-bootstrap react-loader-spinner react-router-dom react-toastify @fortawesome/fontawesome-free @fortawesome/free-brands-svg-icons @fortawesome/free-solid-svg-icons @fortawesome/react-fontawesome

```
# GitHub reads the file and stores the value
# Other jobs can now access it! ✅

# GITHUB_OUTPUT Syntax Explained
```bash
jobs:
  setup-job:                          # job name
    runs-on: ubuntu-latest
    outputs:                          # ← STEP 1: declare outputs
      full_domain: ${{ steps.set_domain.outputs.full_domain }}
      subdomain:   ${{ steps.set_domain.outputs.subdomain }}
    #              ↑           ↑                ↑
    #         reference    step id         key name

    steps:
      - name: Set Domain
        id: set_domain                # ← STEP 2: give step an ID
        run: |
          SUBDOMAIN="reactjs-itp"
          FULL_DOMAIN="reactjs-itp.seang.shop"

          # ← STEP 3: write to GITHUB_OUTPUT
          echo "subdomain=$SUBDOMAIN"     >> $GITHUB_OUTPUT # Part 1 — Write to file
          echo "full_domain=$FULL_DOMAIN" >> $GITHUB_OUTPUT
          #      ↑               ↑
          #    key name        value

```
# How to Read in Another Job
```bash
  other-job:
    needs: [setup-job, ci-job, dns-job]    # ← must declare dependency
    steps:
      - name: Use values
        env:
          # pattern: ${{ needs.JOB_NAME.outputs.OUTPUT_NAME }}
          FULL_DOMAIN: ${{ needs.setup-job.outputs.full_domain }}
          SUBDOMAIN:   ${{ needs.setup-job.outputs.subdomain }}
        run: |
          echo "FULL_DOMAIN = $FULL_DOMAIN"
          echo "SUBDOMAIN   = $SUBDOMAIN"

```

```bash
┌─────────────────────────────────────────────────────────────┐
│                         setup-job                           │
│                      (Machine A) 🖥️                         │
│                                                             │
│  STEP 1: Declare outputs at job level                       │
│  ─────────────────────────────────                         │
│  outputs:                                                   │
│    full_domain: ${{ steps.set_domain.outputs.full_domain }} │
│    subdomain:   ${{ steps.set_domain.outputs.subdomain }}   │
│                                                             │
│  STEP 2: Give step an ID                                    │
│  ───────────────────────                                    │
│    - name: Set Domain                                       │
│      id: set_domain   ← ID                                  │
│                                                             │
│  STEP 3: Write to $GITHUB_OUTPUT                            │
│  ───────────────────────────────                            │
│    echo "subdomain=reactjs-itp"          >> $GITHUB_OUTPUT  │
│    echo "full_domain=reactjs.seang.shop" >> $GITHUB_OUTPUT  │
│                          │                                  │
└──────────────────────────┼──────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────┐
│           GitHub Stores Values  💾        │
│                                          │
│   subdomain   = reactjs-itp              │
│   full_domain = reactjs-itp.seang.shop   │
│                                          │
└──────────────────────────┬───────────────┘
                           │
                           │ passes values
                           │
         ┌─────────────────┼──────────────────┐
         │                 │                  │
         ▼                 ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
│  sonar-job   │  │   ci-job     │  │     cd-job       │
│  (Machine B) │  │  (Machine C) │  │   (Machine D)    │
│              │  │              │  │                  │
│ needs:       │  │ needs:       │  │ needs:           │
│  setup-job   │  │  sonar-job   │  │  setup-job ✅    │
│              │  │              │  │  ci-job    ✅    │
│              │  │ outputs:     │  │  dns-job   ✅    │
│              │  │   tag: abc   │  │                  │
└──────────────┘  └──────┬───────┘  │ env:             │
                         │          │  FULL_DOMAIN:    │
                         │          │  ${{needs.       │
                         │          │  setup-job.      │
                         │          │  outputs.        │
                         │          │  full_domain}} ✅│
                         │          │                  │
                         │          │  SUBDOMAIN:      │
                         │          │  ${{needs.       │
                         │          │  setup-job.      │
                         │          │  outputs.        │
                         │          │  subdomain}}  ✅ │
                         │          │                  │
                         └──────────►  TAG:            │
                                    │  ${{needs.       │
                                    │  ci-job.         │
                                    │  outputs.tag}} ✅│
                                    └──────────────────┘
```

# How to Pass Data to Reusable Workflow
```bash
ci-cd.yml                    dns-cloudflare.yml
(caller)                     (reusable)
    │                             │
    │  with:                      │  inputs:
    │    domain: "seang.shop" ──► │    domain: string
    │    subdomain: "reactjs" ──► │    subdomain: string
    │                             │
```