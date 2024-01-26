---
share: "true"
title: Очистка старых пайплайнов Gitlab
---

Старые пайплайны могут занимать очень много места на диске. И не нужны все эти пайплайны за несколько лет. Из веб-интерфейса Gitlab можно удалять кнопкой только 1 пайплайн, командой через API так же можно удалить пайплайн, но надо знать номер проекта и номер пайплайна:

```bash
curl --header "PRIVATE-TOKEN: <your_access_token>" --request "DELETE" "https://gitlab.example.com/api/v4/projects/номер_проекта/pipelines/номер_пайплайна"
```

{==

Я нашла [скрипт](https://gist.github.com/chrishoerl), который автоматически удаляет пайплайны. Но он имеет ограничение: обрабатывает максимум 100 проектов за 1 раз и удаляет максимум 100 пайплайнов, т.е. скрипт надо перезапускать. Я доработала скрипт [**cleanup-gitlab-pipelines.sh**](https://gist.github.com/anutator/7f24f83db93bf7cc5d159de4fabd77a7#file-cleanup-gitlab-pipelines-sh)

==}

Cкрипт очистит все старые пайплайны. Надо ввести:
- `GITLABURL` — путь к своему GitLab
- `TOKEN` — токен с правами API чтения и записи
- `DELETEBEFORE` — дату, ДО которой удалить все старые пайплайны.

Предварительно установить утилиту jq.

Токен для gitlab.investpalata.tech glpat-xGAXxWvx8vYSgU_neQL_

```sh title="cleanup-gitlab-pipelines.sh"
#!/bin/bash
# Purpose: Bulk-delete GitLab pipelines older than a given date
# Author: github.com/chrishoerl
# New features: github.com/anutator
# GitLab API: v4
# Requirements: jq must be instaled ($ sudo apt install jq)
# API example: https://gitlab.example.com/api/v4/projects
# API example: https://gitlab.example.com/api/v4/projects/<projectid>/pipelines
#
# NOTE: To dryrun script comment line 59.
#
################### FIRST REPLACE VARIABLES WITH YOR VALUES ############
# Define some variables
GITLABURL="https://gitlab.example.com"
TOKEN="" # must have API r/w access
# Pipelines older than this date will be deleted by this script
DELETEBEFORE="2022-08-15" #date range format: yyyy-mm-dd
########################################################################

# List of projects — not working in Gitlab 14 - test in Gitlab 15 (didn't test yet)
# curl --header "PRIVATE-TOKEN: $ACCESSTOKEN" "$GITLABURL/api/v4/project_aliases"

# Get total number of pages, 100 projects per page
# We don't check archived projects - I couldn't delete pipelines from that projects.
PAGES=$(curl -s --head -H "PRIVATE-TOKEN: $TOKEN" "$GITLABURL/api/v4/projects?archived=false&per_page=100" | grep -i x-total-pages | awk '{print $2}' | tr -d '\r\n')

# Add PROJECTID of all projects from every PAGE to project_array
echo "Please wait while I get all PROJECTIDs and add them to the project_array."
for PAGE in $(seq 1 $PAGES); do
project_array+=( `curl -H "PRIVATE-TOKEN: $TOKEN" "$GITLABURL/api/v4/projects?archived=false&per_page=100&page=$PAGE" 2> /dev/null | jq -r .[].id` )
done

echo "Total number of projects (except archived projects): ${#project_array[@]}"

# Now let's work with the project id from the project_array
# COUNTER will show number of Task — from 1 to total number of projects.
COUNTER=1
for PROJECTID in "${project_array[@]}"; do
   # Get project name for project id
   PROJECTNAME=`curl -H "PRIVATE-TOKEN: $TOKEN" "$GITLABURL/api/v4/projects/$PROJECTID" 2> /dev/null | jq -r .path_with_namespace`

   # Find out pipeline IDs of our project which match the date range and write results into pipeline_array
   pipeline_array=( $(curl -H "PRIVATE-TOKEN: $TOKEN" "$GITLABURL/api/v4/projects/$PROJECTID/pipelines?per_page=100&sort=asc&updated_before=${DELETEBEFORE}T23:01:00.000Z" 2> /dev/null | jq -r .[].id) )

   # Total number of pipelines in one iteration (100 max)
   NUMOFPIPES=`echo ${#pipeline_array[@]}`
   
   echo -e "---\nTask $COUNTER: Project $PROJECTNAME (ID:$PROJECTID) has $NUMOFPIPES pipelines to delete."

  # Delete pipelines - 100 maximum per one iteration
  while [ $NUMOFPIPES -gt 0 ]; do
  # Print all pipeline IDs from the array
    for pipelineid in "${pipeline_array[@]}"; do
      echo "Deleting pipeline ID: ${pipelineid}"
      # echo "$GITLABURL/api/v4/projects/$PROJECTID/pipelines/${pipelineid}"
  
      ## ACTIVATE THIS TO START CLEANUP JOB
      ## Create DELETE query for each pipeline matching our date range
      curl -H "PRIVATE-TOKEN: $TOKEN" --request "DELETE" "$GITLABURL/api/v4/projects/$PROJECTID/pipelines/$pipelineid"
    done
   
    pipeline_array=( $(curl -H "PRIVATE-TOKEN: $TOKEN" "$GITLABURL/api/v4/projects/$PROJECTID/pipelines?per_page=100&sort=asc&updated_before=${DELETEBEFORE}T23:01:00.000Z" 2> /dev/null | jq -r .[].id) )
    NUMOFPIPES=`echo ${#pipeline_array[@]}`
    echo -e "---\nTask $COUNTER: Project $PROJECTNAME (ID:$PROJECTID) has $NUMOFPIPES pipelines to delete."
  done
   
  COUNTER=$((COUNTER + 1))

done
```


%%🔐β DmDoWAb8n/1xXSMab3OUpcav4MP5TIp/40CmRKRqvCm2d2VWMj/pHBZJIXcgc+X3mm+/HKz+dxFC0EloiDllBb2/u7yyBtbmXOF7w++iUl2Na2IQVr1/yufCAabdd4SswmdC2Dj+z3YpuXtsRgeTP9dSrTZmwA== 🔐%%
