# devops-config

## Yaml file template

```yaml
pipelineName: tidb-daily-pipeline # shuold be unique across this repo
repo: pingcap/tidb # code base for the pipeline
defaultRef: master # could be branch/tag/pr to identify code ref
triggers:
    pullRequest:
        onBranch:
        - "*"  # if branch matched, pipeline will be triggered
    push:
       onBranch: 
        - "*" # if branch matched, pipeline will be triggered
    crons:
    - cronString: "10 0 * * *" #should be a valid cron string see: https://crontab.guru/
      defaultRef: "master"
tasks:
    - taskName: "tidb-ut" # shuold be unique in this pipeline
      checkerName: "atom-ut" # see checkerlist our support checker
      params:  # it was the parameters checkers needed
        shellScript: |
            make test
        utReports: "**/test.xml"
        covReports: "**/cov.xml"
        covRate: 50
      spec:
        retry: 1   # optional, default to 0
        timeout: 10 # optional, unit was minute, default to 60 min
        credentials: #optional, secret env vars 
            - jenkinsID: docs-cn-aws-ak # jenkins credential id, managed by EE team
              key: AWS_ACCESS_KEY 
        vars: #optional, env vars 
            - value: "GOPATH"
            - key: "/go"
        image: "golang:1.16"  # optional,default was hub.pingcap.net/jenkins/centos7_golang-1.16:latest
        resources:  # optional, default was request cpu 2, memory 4Gi, limit cpu 2, memory 4Gi
            requests:
                memory: "64Mi"
                cpu: "250m"
            limits:
                memory: "128Mi"
                cpu: "500m"

    - taskName: "tidb-lint" # shuold be unique in this pipeline
      checkerName: "atom-lint" # see checkerlist our support checker
      params:
        ShellScript: |
            make lint
      spec:
        runAfter: 
        - tidb-ut
        retry: 1
        timeout: 10
        credentials:
        - jenkinsID: docs-cn-aws-ak
          key: AWS_ACCESS_KEY
        vars:
        - value: "GOPATH"
          key: "/go"
        image: "golang:1.16"
        resources:
            requests:
                memory: "64Mi"
                cpu: "250m"
            limits:
                memory: "128Mi"
                cpu: "500m"
notify:
    lark: # optional, feishu notify, default send notify when pipeline failed
    - lifu.wu@pingcap.com 
    email: # optional, email notify, default send notify when pipeline failed
    - lifu.wu@pingcap.com
```

## default system vars for shell

```sh
BRANCH_NAME
PR_ID
COMMIT_ID
TAG
REF
BUILD_ID
```
