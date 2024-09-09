---
Title:          openEuler SIG Incubation Process Optimization
Type:           Process
Abstract:       SIG incubation
Author:         Xiong Wei <xiongwei888@huawei.com>, Hu Xinwei <shinwell_hu@openeuler.sh>
Status:         Initial
No:             oEEP-0013
Create_time:    2023-08-17
Update_time:    2023-11-21

---

## Motivation/Problem

### Background

Approximately 100 SIGs have been established in the openEuler community. While many SIGs operate effectively, some face challenges in adhering to operational standards.
This proposal aims to optimize the SIG incubation process. On the one hand, we will establish a framework for assessing the operational efficiency of existing SIGs, enabling the Technical Committee (TC) to implement differentiated management strategies. On the other hand, we will  provide clear guidelines and support for newly formed SIGs, facilitating their integration into the community and enabling them to contribute effectively.

### Current Issues

Several challenges have been observed in both existing and historical SIGs within the openEuler community:

1. SIGs operate without clear objectives, resulting in prolonged periods without code commits or community engagement.
2. SIG development lacks exposure within the broader industry, hindering its recognition and potential impact.
3. SIGs function as isolated units, leading to superficial code review processes and suboptimal code quality and architecture.
4. Difficulties navigating community procedures result in code contributions failing to integrate into LTS versions and a lack of real-world validation.
5. Insufficient interaction between developers and users within the community leads to decreased enthusiasm and developer attrition.

### Related SIGs and Responsibilities

- TC: responsible for the overall management of SIG operations and ensuring their collective effectiveness within the community.
- All SIGs: Each SIG maintainer bears responsibility for the effectiveness of their respective SIG operations.

## Solution Description

### SIG Maturity Evaluation Criteria

#### Clear Basic Information

- The SIG name is easily understandable and free from ambiguity.
- The technical domain of the SIG is clearly articulated, encompassing either a horizontal technology layer or a vertical domain.
- The SIG has a well-defined vision outlining its long-term goals and aspirations.

#### Effective Communication Channels

- SIG maintainers have clearly defined responsibilities.
- The SIG establishes comprehensive communication channels, including mailing lists, regular video conferences, and messaging platforms, to ensure that maintainers are always reachable.
- SIG maintainers possess a thorough understanding of tools and processes used to maintain community information.

#### Rigorous Technical Requirements

- SIG maintainers and committers possess comprehensive expertise in their respective domains and dedicate enough time to SIG activities.
- The SIG maintains a clear list of deliverables that align with its technical scope.
- The SIG employs well-defined principles for version selection or planning for all its deliverables.
- The SIG maintains a comprehensive code review checklist that members apply rigorously during pull request (PR) review.
- The SIG conducts thorough analysis and provides clarification for issues flagged during the continuous integration (CI) process.

#### Active Development and Maintenance

- The SIG possesses a roadmap and technical plan that outlines its future direction.
- The SIG demonstrates familiarity with the process requirements by the Release SIG, with contributions included in innovation or LTS versions.
- The SIG provides timely response to and remediation of security vulnerabilities within the defined scope of openEuler's security service level objectives (SLOs).
- The SIG engages actively in addressing issues raised within related repositories.

#### Vibrant External Engagement

- SIG maintainers understand the community operational requirements, effectively utilize the community communication matrix, proactively submit PRs to the community, and engage in discussions with the TC and other SIGs.
- SIG members actively participate in promoting openEuler.
- SIG members engage in community events, including meetups, openEuler Developer Days, and summits.

### SIG Maturity Levels

While the existing **sig-info.yaml** file includes a **mature_level** field, its definition remains unclear. This proposal provides a clear definition for the **mature_level** field:

- **startup**: SIGs with clear basic information but lacking evaluation data for other criteria. The TC can approve the PR for SIG creation, but a dedicated repository should not be established at this stage. The TC will assign a mentor to the SIG within one month.
- **incubate**: SIGs demonstrating progress in establishing effective communication channels and partial fulfillment of other evaluation criteria. Following consultation with the assigned mentor, the TC can approve the transition to **incubate** status, allowing for the creation of a dedicated repository.
- **graduate:** SIGs excelling in all evaluation criteria and capable of independent operation. Upon a collective request from the SIG maintainers and approval from the TC, the SIG can transition to **graduate** status. Maintainers of **graduate** SIGs are eligible to mentor other SIGs.
