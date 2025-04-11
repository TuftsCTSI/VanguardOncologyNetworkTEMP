# Project Management Workflow within Vanguard Group

## General Idea 

[Robert](https://github.com/rtmill) and [Kyle](https://github.com/kzollove) helped to establish an [oncology-focused GitHub project](https://github.com/orgs/OHDSI/projects/13) that contains 100+ issues organized and grouped into relevant oncology categories.

Instead of duplicating those issues here, we have established a 'view' of those issues in the [Vanguard Network Development](https://github.com/orgs/TuftsCTSI/projects/22) project so we can reference and assign them transparently and independently of OncologyWG activities.

Moreover, we plan to extend the project card attributes on the OncologyWG side to include a 'Vanguard Status' categorical indicator, and a 'VG Proposal Ready' boolean flag that we can update from here to sync

The general structure of this PM workflow can be seen below:

![Image](https://github.com/user-attachments/assets/cdc9c984-ac44-4297-9f35-4d2e7b031221)

Briefly, the Vanguard project will contain three core components, separated across respective project views:
1. Outstanding issues in the OncologyWG project
2. New Vanguard-specific issues
3. Status-related issues that track current oncology implementation at Vanguard data sites

## Issue Import Workflow

The issues on the OncologyWG side can be imported programmatically using the [GH Graph API](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/using-the-api-to-manage-projects).

The first step in the process is to pull the unique issue identifiers from the OncologyWG space by first cloning the OncologyWG repository, and then using the gh cli that filters on open issues that have been added to the PM project:

```sh
gh issue list -L 200 --json projectItems --json id --json title --state open | jq -r '.[] | select( .projectItems != []) | .id' >> issueids.txt
```

This command populates a text file `issueids.txt` with each line representing an issue identifier of an issue active in the OncologyWG project space. The next step is to import those issues, using their identifiers, into the Vanguard Network space, via the graphAPI:

```sh
#!/bin/bash

while read LINE; do
 gh api graphql --silent -f query="
  mutation {
    addProjectV2ItemById(input: {projectId: \"PVT_kwDOB6yPrs4A1HAj\" contentId: \"${LINE}\"}) {
      item {
        id
      }
    }
  }"
 echo "Added issue with id $LINE to Vanguard Project!"
 sleep 2
done < issueids.txt
```

Note that by default, new issues in the OncologyWG project will not be added to the VG space. We can setup a GitHub action to do so should new issues be added to that space and tracked regularly (TBD).

## Scrum Structure

Each week, the Vanguard subgroup of developers will meet to assess and assign tickets across the following categories:
1. Ongoing local work not represented in the OncologyWG space
2. Tasks in the OncologyWG space
3. Status and blocking issues for Vanguard Data Sites

We will aim to prioritize those tickets within the small group, track hour estimates, actuals, and assignments, and synchronize output with the OncologyWG with the intention of providing `beta` implementations and associated validations that enable quicker uptake into broader community conventions. 
