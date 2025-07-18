# Github Action
Github action triggers with an Event. An Event could be a Pull Request, Push, Issue (created or Closed), etc...
Events are tied to Workflow and Workflow has a following typical structure:

**Workflow**
```
🔁 Workflow: `.github/workflows/your-workflow.yml`
├── name: "CI/CD Pipeline"                         # Optional name of the workflow
├── on:                                            # Triggering events
│   ├── push
│   └── pull_request
├── env:                                           # Global environment variables (optional)
│   └── NODE_ENV: production
├── defaults:                                      # Default shell settings (optional)
│   └── run:
│       shell: bash
│
├── 🧩 Job1  → Runs on 🖥️ ubuntu-latest (Runner1)
│   ├── name: "Install & Build"
│   ├── runs-on: ubuntu-latest
│   ├── needs: []                                  # Depends on other jobs (optional)
│   ├── if: github.ref == 'refs/heads/main'        # Job-level conditional (optional)
│   └── steps:
│       ├── ⚙️ Step1: uses: actions/checkout@v4
│       ├── ⚙️ Step2: uses: actions/setup-node@v4
│       ├── 🖊️ Step3: run: npm ci
│       └── 🖊️ Step4: run: npm run build
│
└── 🧩 Job2  → Runs on 🖥️ ubuntu-latest (Runner2)
    ├── name: "Test & Deploy"
    ├── runs-on: ubuntu-latest
    ├── needs: Job1                                 # This job runs only after Job1
    └── steps:
        ├── 🖊️ Step1: run: npm test
        ├── ⚙️ Step2: uses: actions/upload-artifact@v4
        ├── 🖊️ Step3: run: echo "Deploying..."
        └── ⚙️ Step4: uses: some/custom-action@v1
```

A workflow must be defined in this folder structure within the repository.
```
📁 .github/
└── 📁 workflows/
    └── 🔁 ci-cd-pipeline.yml   # Workflow file
```

**Notes**
- Steps under a Job always runs in sequence from top to bottom.
- Jobs consumes independent Runners. Hence, they can be configured to run in Parallel or in Sequence. (Default behaviour is Parallel Run)
- A workflow must be defined within a repository. There is no option to define at the organization level.

## Runners
Runners come in 2 flavours (Github-hosted runners or Self-hosted runners).
- **Github-hosted runners** come in 3 types ubuntu, MacOS and Windows. While desiging the workflow, you can define in which type of runners, the Job will run.
- **Self-hosted runners** are setup by you so you can define which OS you want to have on it.

## Workflow Syntax


## Advanced Workflow Structure
```md
🔁 Workflow: `.github/workflows/your-workflow.yml`
├── name: "CI/CD Pipeline"                         # Optional workflow name
├── on:                                            # Triggering events
│   ├── push:
│   │   ├── branches: [main, release/*]            # Branch filters
│   │   └── paths-ignore:                           # Ignore paths from triggering
│   │       └── ["docs/**", "*.md"]
│   ├── pull_request:
│   │   ├── branches: [main]
│   │   └── paths: ["src/**", "package.json"]
│   └── workflow_dispatch                           # Manual trigger
├── concurrency:                                    # Prevent parallel runs for the same ref
│   ├── group: "ci-cd-${{ github.ref }}"           
│   └── cancel-in-progress: true
├── permissions:                                   # Minimal permissions for workflow token
│   ├── contents: read
│   └── actions: write
├── env:                                           # Global environment variables
│   └── NODE_ENV: production
├── defaults:                                      # Default shell options
│   └── run:
│       shell: bash
│       working-directory: ./src
│
├── 🧩 Job1  → Runs on 🖥️ ubuntu-latest (Runner1)
│   ├── name: "Install & Build"
│   ├── runs-on: ubuntu-latest
│   ├── timeout-minutes: 30                         # Job timeout
│   ├── strategy:                                  # Matrix build example
│   │   └── matrix:
│   │       ├── node-version: [16.x, 18.x]
│   │       └── os: [ubuntu-latest, windows-latest]
│   ├── outputs:                                   # Outputs to pass to other jobs
│   │   └── build_artifact_path: ${{ steps.build.outputs.artifact-path }}
│   ├── steps:
│   │   ├── ⚙️ Step1: uses: actions/checkout@v4
│   │   ├── ⚙️ Step2: uses: actions/setup-node@v4
│   │   ├── 🖊️ Step3: run: npm ci
│   │   ├── 🖊️ Step4: id: build
│   │   │           run: |
│   │   │             npm run build
│   │   │             echo "::set-output name=artifact-path::./build"
│   │   └── ⚙️ Step5: uses: actions/upload-artifact@v3
│   │               with:
│   │                 name: build-artifact
│   │                 path: ${{ steps.build.outputs.artifact-path }}
│
└── 🧩 Job2  → Runs on 🖥️ ubuntu-latest (Runner2)
    ├── name: "Test & Deploy"
    ├── runs-on: ubuntu-latest
    ├── needs: Job1                                 # Depends on Job1
    ├── if: github.ref == 'refs/heads/main'        # Conditional job run
    ├── timeout-minutes: 20
    ├── steps:
    │   ├── 🖊️ Step1: run: npm test
    │   ├── ⚙️ Step2: uses: actions/download-artifact@v3
    │   │         with:
    │   │           name: build-artifact
    │   ├── 🖊️ Step3: run: echo "Deploying..."
    │   └── ⚙️ Step4: uses: some/custom-action@v1
```

